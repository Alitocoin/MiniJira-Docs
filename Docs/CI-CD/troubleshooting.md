# Troubleshooting — Pipelines CI/CD

Guía de resolución de problemas comunes. Para cada problema se indica dónde ver el error real y qué hacer.

---

## Pipeline falla en `npm ci`

**Síntoma:** el paso "Instalar dependencias (npm ci)" falla con exit code != 0.

**Causas comunes:**

1. **Lockfile desactualizado:** `package.json` y `package-lock.json` están desfasados. Sucede cuando alguien editó `package.json` manualmente sin correr `npm install` después.

   Solución:
   ```bash
   # En MiniJira-FE
   rm -rf node_modules
   npm install          # regenera package-lock.json
   git add package-lock.json
   git commit -m "chore: regenerar package-lock.json"
   ```

2. **Dependencia que no resuelve:** un paquete fue removido del registro de npm (raro pero ocurre con paquetes privados o deprecados abruptamente).

   Identificar cuál paquete falla leyendo el log del paso "npm ci" en GitHub Actions. El error menciona el nombre del paquete.

3. **Conflicto de versiones de Node:** si `package.json` tiene un campo `engines.node` que no es compatible con Node 20.

   Solución: actualizar el campo `engines` o coordinar con el equipo el cambio de versión de Node en el pipeline.

---

## Pipeline falla en lint

**Síntoma:** el paso "Lint (ESLint)" falla.

**Cómo ver los errores:**

En el log del pipeline (GitHub Actions), el paso imprime cada archivo con error, el número de línea y la regla violada. Por ejemplo:
```
src/components/TaskCard.tsx
  15:5  error  'useEffect' is defined but never used  no-unused-vars
```

**Resolución:**

1. Correr lint localmente para ver todos los errores de una vez:
   ```bash
   npm run lint
   ```

2. Para auto-fix de errores corregibles automáticamente:
   ```bash
   npm run lint -- --fix
   ```
   No todos los errores son auto-corregibles: los que requieren lógica (variables no usadas que deberían eliminarse, imports incorrectos) necesitan intervención manual.

3. Si una regla específica genera demasiado ruido y no aplica al proyecto, coordinar con el equipo si conviene deshabilitarla en `eslint.config.js`. No deshabilitar reglas de seguridad (ej. `no-eval`).

---

## OWASP tarda mucho en la primera ejecución

**Síntoma:** el paso "OWASP Dependency Check" tarda entre 5 y 15 minutos en la primera ejecución del pipeline.

**Causa:** OWASP Dependency Check descarga la base de datos NVD completa del NIST (~300 MB) la primera vez que se ejecuta. En GitHub Actions, el cache de Maven (`~/.m2/repository`) no incluye esta base de datos por defecto.

**Comportamiento esperado:**
- Primera ejecución: lenta (~5-15 min según velocidad de la conexión del runner)
- Ejecuciones siguientes en el mismo runner/cache: usan la DB cacheada, son significativamente más rápidas

**Si el paso falla con error de conexión a NVD:**
La NVD tiene límites de rate en su API. GitHub Actions puede verse afectado si muchos proyectos consultan la API simultáneamente. En ese caso, el pipeline puede reintentarse manualmente (botón "Re-run failed jobs" en GitHub).

**Optimización recomendada (no implementada aún):**
Agregar la ruta de la base de datos OWASP al cache de GitHub Actions:
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository/org/owasp
    key: owasp-nvd-${{ runner.os }}-${{ hashFiles('**/pom.xml') }}
```
Ver `execution-report.md` sección Recomendaciones para más contexto.

---

## npm audit falla — identificar la dependencia vulnerable

**Síntoma:** el paso "Audit de seguridad (npm audit)" falla con exit code != 0.

**Cómo ver qué dependencia tiene la vulnerabilidad:**

```bash
# En MiniJira-FE, ejecutar localmente
npm audit

# Output más detallado en JSON
npm audit --json | jq '.vulnerabilities'
```

El output de `npm audit` lista:
- Nombre del paquete vulnerable
- Versión afectada
- Severidad (high/critical)
- CVE ID
- Dependencia que lo trajo (si es transitiva: la cadena completa)
- Si existe un fix disponible

Si la vulnerabilidad está en una dependencia transitiva (no en las que declaraste directamente), `npm audit` te indica el paquete "padre" que la trajo. El fix puede requerir actualizar ese paquete padre a una versión que dependa de una versión segura del transitivo.

Ver `security-scan.md` para opciones de corrección.

---

## `mvn verify` falla

**Síntoma:** el paso "Verificación (mvn verify)" falla después de que "Tests unitarios (mvn clean test)" pasó.

**Causas comunes:**

1. **Integration tests que fallan:** `mvn verify` incluye la fase `integration-test`. Si el proyecto tiene tests de integración que requieren una base de datos o servicio externo, pueden fallar en CI si no están mockeados.

   Para ver el stack trace completo:
   ```bash
   # Localmente, sin --no-transfer-progress para ver todo el output
   mvn verify
   ```

   Los logs de Maven muestran el test que falló y el stack trace. Buscar líneas como:
   ```
   [ERROR] Tests run: 5, Failures: 1, Errors: 0, Skipped: 0
   ```

2. **Error de packaging:** `mvn verify` también empaqueta el JAR. Si hay un recurso faltante o un plugin mal configurado, puede fallar aquí y no en `test`.

3. **Checkstyle o PMD configurados:** si el proyecto tiene plugins de análisis estático en la fase `verify`, pueden fallar aquí. Revisar `pom.xml` en la sección `<plugins>`.

**Para ver el stack trace de un test específico:**
```bash
mvn verify -pl . -Dtest=NombreDelTestQueFlló -e
```

---

## Docker Compose no levanta — conflicto de puertos

**Síntoma:** `docker compose up` falla con error similar a:
```
Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use
```

**Causa:** otro proceso está usando el puerto que intenta exponer el container.

**Diagnostico:**

```bash
# Ver qué proceso usa el puerto 8080 (backend)
lsof -i :8080

# Ver qué proceso usa el puerto 5432 (postgres)
lsof -i :5432

# Ver qué proceso usa el puerto 3000 (frontend)
lsof -i :3000
```

**Resoluciones:**

Opción A — matar el proceso que ocupa el puerto:
```bash
kill -9 <PID>
```

Opción B — cambiar el puerto del lado del host en el `docker-compose.yml` local (no commitear este cambio):
```yaml
ports:
  - "8081:8080"   # usa 8081 en el host en vez de 8080
```

Si el puerto ocupado es `5432`, probablemente tenés una instancia local de PostgreSQL corriendo. Podés detenerla:
```bash
# Linux
sudo systemctl stop postgresql

# Mac con Homebrew
brew services stop postgresql
```

---

## El paso de lint se saltea cuando no debería

**Síntoma:** el pipeline imprime "Lint configuration not found. Documented as pending." pero el archivo de configuración de ESLint existe.

**Causa:** el pipeline busca específicamente estos nombres de archivo:
```
eslint.config.js
.eslintrc
.eslintrc.js
.eslintrc.json
.eslintrc.yml
```

Si usás un nombre diferente (ej. `eslint.config.mjs`, `eslint.config.cjs`) no lo detecta.

**Solución:** renombrar el archivo a uno de los nombres esperados, o actualizar el step `check-lint` en `ci-frontend.yml` para incluir el nombre adicional.
