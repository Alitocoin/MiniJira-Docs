# Scans de seguridad

El sistema CI/CD de MiniJira implementa dos mecanismos de detección de vulnerabilidades en dependencias: uno para el frontend (npm audit) y otro para el backend (OWASP dependency-check).

---

## npm audit — Frontend

### Qué es

`npm audit` compara las dependencias listadas en `package-lock.json` contra el registro de vulnerabilidades de npm (alimentado por bases de datos como la NVD). Genera un reporte de vulnerabilidades encontradas.

### Configuración en el pipeline

```bash
npm audit --audit-level=high
```

El flag `--audit-level=high` hace que el comando retorne exit code != 0 (y falle el pipeline) solo si hay vulnerabilidades de nivel `high` o `critical`. Los niveles `low` y `moderate` se reportan pero no bloquean.

### Escala de severidad npm

| Nivel | Descripción | Bloquea el pipeline |
|---|---|---|
| `critical` | Explotable remotamente, impacto severo | Si |
| `high` | Impacto significativo, explotable | Si |
| `moderate` | Impacto limitado o requiere condiciones específicas | No |
| `low` | Impacto mínimo | No |

### Qué significa que falle

Si el pipeline falla en npm audit, significa que al menos una dependencia declarada en `package-lock.json` tiene una vulnerabilidad registrada de nivel `high` o `critical`. Esto no implica necesariamente que el proyecto sea explotable — puede ser una vulnerabilidad en una dependencia de desarrollo que no llega al build de producción.

### Cómo interpretar el output

```
# Ver detalle completo de vulnerabilidades
npm audit

# Ver solo las de nivel high y critical
npm audit --audit-level=high

# Output en formato JSON (para parsing)
npm audit --json
```

El output lista: nombre del paquete, versión afectada, severidad, descripción de la CVE y la ruta de dependencia (directo o transitivo).

### Cómo corregir

**Opción 1 — Fix automático (cuando existe una versión segura):**
```bash
npm audit fix
```
Actualiza las dependencias afectadas a versiones seguras sin romper semver. Después de esto, commitear el `package-lock.json` actualizado.

**Opción 2 — Fix forzado (puede romper compatibilidad):**
```bash
npm audit fix --force
```
Usarlo con precaución: puede subir versiones mayor que rompan la API.

**Opción 3 — Actualización manual:**
```bash
npm update <nombre-paquete>
# o directamente en package.json cambiar la versión
```

**Opción 4 — Si el paquete vulnerado es transitivo:**
Usá `overrides` en `package.json` para forzar una versión segura de la dependencia transitiva:
```json
{
  "overrides": {
    "nombre-paquete-transitivo": ">=version-segura"
  }
}
```

### Consideración para dependencias de desarrollo

`npm audit` por defecto escanea todas las dependencias, incluyendo las de desarrollo (`devDependencies`). Si una vulnerabilidad está en un paquete de dev que no llega al bundle de producción, puede considerarse aceptable omitirla:

```bash
npm audit --omit=dev
```

Si el equipo decide adoptar esto, actualizar el pipeline en `ci-frontend.yml`.

---

## OWASP Dependency Check — Backend

### Qué es

OWASP Dependency Check es una herramienta open source que analiza las dependencias Java (JARs) declaradas en `pom.xml` y las compara contra la base de datos NVD (National Vulnerability Database) del NIST. Genera un reporte HTML detallado.

### Configuración en el pipeline

```bash
mvn org.owasp:dependency-check-maven:check \
  -DfailBuildOnCVSS=7 \
  -DsuppressionFile="" \
  --no-transfer-progress
```

### La escala CVSS

CVSS (Common Vulnerability Scoring System) es un estándar de la industria para cuantificar la severidad de vulnerabilidades. Va de 0.0 a 10.0:

| Rango CVSS | Severidad | Bloquea el pipeline |
|---|---|---|
| 9.0 - 10.0 | Critical | Si (>= 7) |
| 7.0 - 8.9 | High | Si (>= 7) |
| 4.0 - 6.9 | Medium | No |
| 0.1 - 3.9 | Low | No |
| 0.0 | None | No |

### Por qué threshold = 7

El umbral `failBuildOnCVSS=7` bloquea el pipeline para vulnerabilidades `High` y `Critical`. Las vulnerabilidades `Medium` (CVSS 4.0-6.9) generan advertencias en el reporte pero no bloquean el desarrollo — generalmente requieren condiciones específicas de explotación o tienen impacto limitado.

Este es el balance recomendado por OWASP para equipos que adoptan seguridad incremental: no tan permisivo que deje pasar riesgos reales, no tan estricto que bloquee el trabajo por vulns teóricas de bajo impacto.

### Cómo interpretar el reporte HTML

El reporte `target/dependency-check-report.html` está disponible como artefacto descargable en GitHub Actions (nombre: `owasp-dependency-check-report`).

El reporte contiene para cada dependencia afectada:
- Nombre y versión del JAR escaneado
- CVE ID (ej. `CVE-2023-12345`)
- Puntaje CVSS y severidad
- Descripción de la vulnerabilidad
- Referencias a avisos de seguridad
- Evidencia de por qué OWASP asoció el JAR a esa CVE

### Cómo suprimir falsos positivos

OWASP puede marcar dependencias como vulnerables cuando la evidencia de matching es débil (ej. un JAR con nombre similar a otro vulnerable). Estos son falsos positivos.

Para suprimirlos, creá un archivo `dependency-check-suppressions.xml` en la raíz del proyecto backend:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
  <suppress>
    <notes>Falso positivo: el JAR solo comparte nombre con la librería vulnerable.</notes>
    <gav regex="true">^com\.example:mi-libreria:.*$</gav>
    <cve>CVE-2023-12345</cve>
  </suppress>
</suppressions>
```

Luego referenciarlo en el pipeline:
```bash
mvn org.owasp:dependency-check-maven:check \
  -DfailBuildOnCVSS=7 \
  -DsuppressionFile=dependency-check-suppressions.xml \
  --no-transfer-progress
```

Actualizá también `ci-backend.yml` para pasar el mismo flag.

Importante: cada supresión debe estar documentada con `<notes>` explicando por qué es un falso positivo. No suprimas CVEs sin revisar el reporte primero.

### Actualizar una dependencia vulnerable

Si OWASP reporta una vulnerabilidad real en una dependencia Maven, el proceso es:

1. Identificar el artefacto en el reporte (`groupId:artifactId:version`)
2. Buscar si existe una versión parcheada en Maven Central
3. Actualizar la versión en `pom.xml`
4. Correr `mvn verify` localmente para asegurarte que no rompiste nada
5. Pushear y verificar que el pipeline pasa
