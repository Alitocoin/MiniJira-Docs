# Pipeline CI — MiniJira Backend

Archivo de workflow: `MiniJira-BE/.github/workflows/ci-backend.yml`

## Trigger

El pipeline se activa en:
- `push` a la branch `test-05-cicd_v01`
- `pull_request` hacia la branch `test-05-cicd_v01`

Runner: `ubuntu-latest`
Job: `build-and-test — Build, Test y Security — MiniJira BE`

## Los 8 pasos del pipeline

### Paso 1 — Checkout del código
```yaml
uses: actions/checkout@v4
```
Clona el repositorio en el runner.

**Falla el pipeline:** si el checkout falla.

---

### Paso 2 — Setup Java 17 (Temurin)
```yaml
uses: actions/setup-java@v4
with:
  java-version: '17'
  distribution: 'temurin'
```
Instala Java 17 de la distribución Eclipse Temurin (anteriormente AdoptOpenJDK). Temurin es open source, de soporte a largo plazo y la distribución recomendada por defecto en GitHub Actions para Java.

**Falla el pipeline:** si GitHub Actions no puede resolver la distribución o versión.

---

### Paso 3 — Cache de dependencias Maven
```yaml
uses: actions/cache@v4
with:
  path: ~/.m2/repository
  key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
  restore-keys: |
    ${{ runner.os }}-maven-
```
Cachea el repositorio local de Maven (`~/.m2/repository`) usando el hash del `pom.xml` como clave. En runs subsiguientes donde el `pom.xml` no cambió, Maven no descarga dependencias de internet.

Nota: el cache NO incluye la base de datos NVD de OWASP. La primera ejecución del OWASP check descarga ~300 MB adicionales. Ver `troubleshooting.md` para más detalle.

**No falla el pipeline:** un cache miss solo significa descarga completa.

---

### Paso 4 — Tests unitarios
```bash
mvn clean test --no-transfer-progress
```
Ejecuta los tests unitarios con JUnit + Mockito. El flag `--no-transfer-progress` elimina el ruido de descarga de artefactos en los logs de CI. `clean` garantiza un estado limpio antes de compilar.

Los tests viven en `src/test/java/`.

**Falla el pipeline:** si algún test falla o hay errores de compilación.

---

### Paso 5 — Verificación integrada
```bash
mvn verify --no-transfer-progress
```
Corre el ciclo completo de Maven hasta la fase `verify`: incluye compilación, tests unitarios e integration tests (si están configurados), y empaquetado. Es más amplio que `mvn test`.

**Falla el pipeline:** si cualquier fase del ciclo de verify falla.

---

### Paso 6 — OWASP Dependency Check
```bash
mvn org.owasp:dependency-check-maven:check \
  -DfailBuildOnCVSS=7 \
  -DsuppressionFile="" \
  --no-transfer-progress
```
Escanea todas las dependencias declaradas en `pom.xml` contra la base de datos NVD (National Vulnerability Database) del NIST. Genera un reporte HTML en `target/dependency-check-report.html`.

**Threshold:** `failBuildOnCVSS=7` — el pipeline falla si se detecta alguna vulnerabilidad con puntaje CVSS >= 7.0 (severidad `High` o `Critical`). Vulnerabilidades con CVSS < 7 (Medium, Low) generan advertencias en el reporte pero no bloquean.

**Falla el pipeline:** si alguna dependencia tiene una vulnerabilidad con CVSS >= 7.

Ver `security-scan.md` para entender la escala CVSS y cómo manejar falsos positivos.

---

### Paso 7 — Upload del reporte OWASP
```yaml
uses: actions/upload-artifact@v4
if: always()
with:
  name: owasp-dependency-check-report
  path: target/dependency-check-report.html
  retention-days: 7
```
Sube el reporte HTML de OWASP como artefacto descargable. Se ejecuta **siempre** (`if: always()`), incluyendo cuando el paso 6 falla — precisamente para que el equipo pueda revisar qué dependencia causó el fallo.

---

### Paso 8 — Resumen del pipeline
```bash
if: always()
```
Imprime en el log de GitHub Actions un resumen con las configuraciones utilizadas (Java 17, Temurin, umbrales de OWASP). Siempre se ejecuta.

## Qué falla vs. qué no falla el pipeline

| Paso | Bloquea el pipeline |
|---|---|
| Checkout | Si |
| Setup Java 17 | Si |
| mvn clean test | Si |
| mvn verify | Si |
| OWASP (CVSS >= 7) | Si |
| Cache de Maven | No |
| Upload de reporte OWASP | No |
| Resumen | No |

## Configuración Java/Maven

- **Java:** 17, distribución Temurin
- **Maven:** usa el `mvnw` del proyecto o Maven del sistema
- **Flag `--no-transfer-progress`:** suprime los logs de descarga de artefactos Maven en CI, haciendo los logs más legibles
- **Cache:** `~/.m2/repository` keyed por `pom.xml` hash

## Artefactos generados

- **Nombre:** `owasp-dependency-check-report`
- **Contenido:** archivo HTML con el reporte de vulnerabilidades de dependencias
- **Retención:** 7 dias
- **Descarga:** desde la pestaña "Artifacts" del run en GitHub Actions

El reporte HTML lista cada dependencia escaneada, las CVEs encontradas, el puntaje CVSS y recomendaciones de mitigación.
