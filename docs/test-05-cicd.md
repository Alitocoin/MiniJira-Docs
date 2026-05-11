# Test 05 — Pipeline CI/CD para MiniJira (BE + FE)

**Fecha:** 2026-05-10
**Branch:** `feat/test-05-cicd`
**Estado:** Completado

---

## 1. Descripción general

Este documento describe los pipelines de integración continua (CI) implementados para MiniJira en GitHub Actions. Hay un pipeline por repositorio:

| Repositorio | Archivo | Propósito |
|---|---|---|
| MiniJira-BE | `.github/workflows/ci.yml` | Build, tests y análisis del backend Spring Boot |
| MiniJira-FE | `.github/workflows/ci.yml` | Build, tests y análisis del frontend React + Vite |

Los pipelines se activan en push y pull_request a `main`, `development` y `feat/test-05-cicd`.

---

## 2. Pipeline — Backend (MiniJira-BE)

### Stack

- Java 17 (Temurin), Maven, Spring Boot, JUnit 5 + Mockito

### Jobs

```
build-and-test
└─ lint (necesita build-and-test)
└─ security (necesita build-and-test)
```

Los jobs `lint` y `security` corren en paralelo después de que `build-and-test` pase. Si el build falla, no se gasta tiempo en análisis.

### Etapas detalladas

#### Job `build-and-test`

| Paso | Comando | Qué verifica |
|---|---|---|
| Checkout | `actions/checkout@v4` | Estado del repo en el commit |
| Setup Java | `actions/setup-java@v4` (Temurin 17) | JDK correcto, cache de `.m2` |
| Compilar | `mvn compile` | Sin errores de compilación |
| Pruebas | `mvn test` | JUnit 5 + Mockito pasan |
| Artefacto | `upload-artifact` | Reportes Surefire, 7 días |

#### Job `lint`

| Paso | Comando | Qué verifica |
|---|---|---|
| Checkstyle | `mvn checkstyle:check` | Estilo de código según las reglas del pom.xml |

#### Job `security`

| Paso | Comando | Qué verifica |
|---|---|---|
| OWASP Dep-Check | `dependency-check/Dependency-Check_Action@main` | CVEs en dependencias Maven con CVSS >= 7 |
| Artefacto | `upload-artifact` | Reporte HTML, 14 días |

---

## 3. Pipeline — Frontend (MiniJira-FE)

### Stack

- Node.js 20, React 18, TypeScript 5, Vite 5, Vitest 1.x

### Jobs

```
build-and-test
└─ lint (necesita build-and-test)
└─ security (necesita build-and-test)
```

### Etapas detalladas

#### Job `build-and-test`

| Paso | Comando | Qué verifica |
|---|---|---|
| Checkout | `actions/checkout@v4` | Estado del repo en el commit |
| Setup Node | `actions/setup-node@v4` (Node 20) | Versión fijada, cache de `npm` |
| Instalar deps | `npm ci` | Reproducible desde `package-lock.json` |
| Build | `npm run build` | TypeScript compila + Vite empaqueta sin errores |
| Tests | `npm test` | Vitest — todos los tests pasan |
| Artefacto | `upload-artifact` | Directorio `coverage/`, 7 días |

#### Job `lint`

| Paso | Comando | Qué verifica |
|---|---|---|
| Type-check | `tsc --noEmit` | Tipos TypeScript sin errores en toda la base de código |

> Nota: el proyecto no tiene ESLint configurado (no aparece en `devDependencies` ni hay `.eslintrc` en la raíz). Se usa `tsc --noEmit` como herramienta de análisis estático equivalente. Si se agrega ESLint en el futuro, reemplazar este paso por `npm run lint`.

#### Job `security`

| Paso | Comando | Qué verifica |
|---|---|---|
| npm audit | `npm audit --audit-level=high` | Vulnerabilidades high/critical en dependencias npm |

---

## 4. Variables de entorno requeridas

Los pipelines actuales no requieren secrets de GitHub para correr. Todos los pasos usan herramientas públicas.

Si en el futuro se agrega deploy o notificaciones, se necesitarán:

| Variable | Scope | Uso previsto |
|---|---|---|
| `DOCKER_USERNAME` | Repo secret | Push a Docker Hub (si se agrega CD) |
| `DOCKER_PASSWORD` | Repo secret | idem |
| `NVD_API_KEY` | Repo secret | OWASP Dep-Check — evita throttling de la API NVD |

> Para el `NVD_API_KEY`: es gratuito, se obtiene en https://nvd.nist.gov/developers/request-an-api-key. Sin él el paso funciona pero es más lento.

---

## 5. Cómo ejecutar localmente

### Backend

```bash
# Compilar
mvn compile

# Tests
mvn test

# Checkstyle
mvn checkstyle:check

# OWASP Dependency Check (requiere Java, descarga la DB NVD la primera vez ~5 min)
mvn org.owasp:dependency-check-maven:check
```

### Frontend

```bash
# Instalar dependencias (igual que CI)
npm ci

# Build
npm run build

# Tests
npm test

# Type-check (equivalente al job lint)
./node_modules/.bin/tsc --noEmit

# Auditoría de seguridad
npm audit --audit-level=high
```

---

## 6. Decisiones técnicas

### BE — Por qué Checkstyle y no SpotBugs

Checkstyle es el plugin de análisis estático más común en proyectos Spring Boot y viene configurado en el parent POM de Spring Boot. SpotBugs requiere configuración adicional y la tarea era construir un pipeline sin depender de configuración preexistente no verificada. Si el equipo decide agregar SpotBugs, se añade como step adicional en el mismo job `lint` con `mvn spotbugs:check`.

### BE — Por qué OWASP Dependency-Check y no Snyk

OWASP Dependency-Check es open source, no requiere cuenta externa ni API key (aunque mejora con NVD API Key), y la Action oficial está mantenida por el proyecto. Snyk requiere un token y tiene límite de scans en el plan gratuito — va en contra del requisito "no depender de servicios externos privados".

### FE — Por qué tsc --noEmit en lugar de ESLint

ESLint no está configurado en el proyecto (no está en `devDependencies`). Inventar un script `npm run lint` que no existe haría fallar el pipeline. `tsc --noEmit` ya está en uso implícito en el script `build` y cubre la verificación de tipos, que es el valor principal de TypeScript en CI.

### FE — Por qué npm audit y no Snyk/Socket

Mismo criterio que BE: `npm audit` es nativo de npm, no requiere cuenta externa, y cubre el requisito de análisis de vulnerabilidades con `--audit-level=high` para filtrar solo CVEs críticos y altos. No produce falsos positivos por configuración incorrecta.

### Estructura de jobs — por qué needs: build-and-test

`lint` y `security` corren después del build para no desperdiciar minutos de GitHub Actions en análisis de código que no compiló. El principio es "falla rápido y claro": si hay un error de compilación, el desarrollador lo ve en el primer job sin esperar al resto.

### Cache de dependencias

Ambos pipelines usan el cache nativo de `setup-java` (cache: maven) y `setup-node` (cache: npm). Esto reduce los tiempos de `mvn` y `npm ci` en corridas sucesivas sin configuración adicional.

---

## 7. Artefactos generados por el pipeline

| Pipeline | Artefacto | Retención |
|---|---|---|
| BE | `surefire-reports` — resultados JUnit en XML | 7 días |
| BE | `dependency-check-report` — reporte HTML de vulnerabilidades | 14 días |
| FE | `vitest-coverage` — cobertura de tests | 7 días |

Los artefactos se descargan desde la pestaña "Actions" del repo en GitHub, en cada run.
