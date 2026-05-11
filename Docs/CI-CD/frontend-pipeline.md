# Pipeline CI — MiniJira Frontend

Archivo de workflow: `MiniJira-FE/.github/workflows/ci-frontend.yml`

## Trigger

El pipeline se activa en:
- `push` a la branch `test-05-cicd_v01`
- `pull_request` hacia la branch `test-05-cicd_v01`

Runner: `ubuntu-latest`
Job: `build-and-test — Build, Test y Audit — MiniJira FE`

## Los 10 pasos del pipeline

### Paso 1 — Checkout del código
```yaml
uses: actions/checkout@v4
```
Clona el repositorio en el runner. Sin opciones adicionales: clona el commit que disparó el evento.

**Falla el pipeline:** si el checkout falla (raro, indica problema de permisos en el repo).

---

### Paso 2 — Setup Node.js 20
```yaml
uses: actions/setup-node@v4
with:
  node-version: '20'
```
Instala Node.js 20 en el runner. Versión fija para reproducibilidad.

**Falla el pipeline:** si GitHub Actions no puede resolver la versión solicitada.

---

### Paso 3 — Cache de dependencias npm
```yaml
uses: actions/cache@v4
with:
  path: ~/.npm
  key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
  restore-keys: |
    ${{ runner.os }}-node-
```
Cachea `~/.npm` usando el hash de `package-lock.json` como clave. Si el lockfile no cambió desde el último run, las dependencias se restauran del cache y `npm ci` tarda segundos en vez de minutos.

**No falla el pipeline:** un cache miss solo significa que npm descarga desde internet.

---

### Paso 4 — Instalar dependencias (`npm ci`)
```yaml
run: npm ci
```
Instala exactamente las versiones del `package-lock.json`. Más estricto que `npm install`: falla si hay discrepancias entre `package.json` y el lockfile.

**Falla el pipeline:** si `npm ci` retorna código de error. Causa más común: lockfile desactualizado o dependencia que no resuelve.

---

### Paso 5 — Lint (condicional)

**5a — Verificación de config de lint** (id: `check-lint`):
Busca cualquiera de estos archivos: `eslint.config.js`, `.eslintrc`, `.eslintrc.js`, `.eslintrc.json`, `.eslintrc.yml`. Si encuentra al menos uno, exporta `lint_exists=true`.

**5b — Ejecución de lint** (solo si `lint_exists == 'true'`):
```bash
npm run lint
```

Estado actual: `eslint.config.js` existe en MiniJira-FE. El pipeline **corre lint**.

Si no existiera configuración de lint: imprime `"Lint configuration not found. Documented as pending."` y continua sin fallar.

**Falla el pipeline:** solo si lint existe Y reporta errores (exit code != 0).

---

### Paso 6 — Tests unitarios (condicional)

**6a — Verificación de existencia de tests** (id: `check-tests`):
```bash
TEST_COUNT=$(find src -name "*.test.*" -o -name "*.spec.*" 2>/dev/null | wc -l)
```
Si encuentra al menos 1 archivo, exporta `tests_exist=true`.

**6b — Ejecución de tests** (solo si `tests_exist == 'true'`):
```bash
npm run test -- --run
```
El flag `--run` ejecuta Vitest en modo no-interactivo (sin watch).

Estado actual: existen ~10 archivos en `src/__tests__/`. El pipeline **corre los tests**.

Si no existieran tests: imprime `"Tests not found. Documented as pending."` y continua sin fallar.

**Falla el pipeline:** solo si los tests existen Y alguno falla.

---

### Paso 7 — Build de producción
```bash
npm run build
```
Corre Vite en modo producción. Genera artefactos en `dist/`. Este paso no tiene condicional: siempre se ejecuta.

**Falla el pipeline:** si el build falla (error de TypeScript, import roto, etc.). Es un paso bloqueante sin condición de skip.

---

### Paso 8 — Audit de seguridad (condicional)

**8a — Verificación de lockfile** (id: `check-audit`):
Comprueba si existe `package-lock.json`. Si existe, exporta `audit_ready=true`.

**8b — Ejecución de audit** (solo si `audit_ready == 'true'`):
```bash
npm audit --audit-level=high
```
Reporta vulnerabilidades de nivel `high` y `critical`. Niveles `moderate` y `low` no bloquean.

Estado actual: `package-lock.json` presente. El pipeline **corre npm audit**.

Si no existiera lockfile: imprime `"Security audit tooling not configured. Documented as pending."` y continua sin fallar.

**Falla el pipeline:** si npm audit detecta al menos una vulnerabilidad de nivel `high` o `critical`.

---

### Paso 9 — Upload de artefacto
```yaml
uses: actions/upload-artifact@v4
if: always()
with:
  name: frontend-dist
  path: dist/
  retention-days: 7
```
Sube el contenido de `dist/` como artefacto descargable desde la UI de GitHub Actions. Se ejecuta **siempre** (incluso si pasos anteriores fallaron), gracias a `if: always()`. Retención: 7 días.

**No falla el pipeline:** si `dist/` no existe (porque el build falló), el artefacto queda vacío.

---

### Paso 10 — Resumen del pipeline
```bash
if: always()
```
Imprime en el log de GitHub Actions un resumen con el estado de cada componente verificado (lint, tests, audit). Siempre se ejecuta para tener visibilidad incluso en runs fallidos.

## Qué falla vs. qué no falla el pipeline

| Paso | Bloquea el pipeline |
|---|---|
| npm ci | Si |
| Lint (cuando existe config) | Si |
| Tests (cuando existen) | Si |
| Build | Si |
| npm audit (cuando existe lockfile) | Si |
| Cache de npm | No |
| Upload de artefacto | No |
| Resumen | No |

## Artefactos generados

- **Nombre:** `frontend-dist`
- **Contenido:** directorio `dist/` con el build de producción (HTML, JS, CSS)
- **Retención:** 7 dias
- **Descarga:** desde la pestaña "Artifacts" del run en GitHub Actions

## Comportamiento cuando los componentes no existen

El pipeline fue diseñado para no romper proyectos que aún no tienen lint o tests configurados. Si algún componente no existe, el paso se marca como "documentado como pendiente" y el pipeline continúa. Esto permite adoptar CI incrementalmente sin bloquear al equipo.
