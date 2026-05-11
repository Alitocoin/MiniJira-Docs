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

```yaml
name: MiniJira BE CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '17', distribution: 'temurin' }
      - name: Cache Maven
        uses: actions/cache@v4
      - name: Build & Test
        run: mvn --no-transfer-progress verify
      - name: Checkstyle
        run: mvn checkstyle:check
      - name: OWASP Dependency Check
        run: mvn org.owasp:dependency-check-maven:check
```

---

## 3. Pipeline — Frontend (MiniJira-FE)

### Stack

- Node 20, npm, Vite, Vitest, TypeScript

### Jobs

```yaml
name: MiniJira FE CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - name: Cache npm
        uses: actions/cache@v4
      - run: npm ci
      - run: npm test -- --run
      - run: npx tsc --noEmit
      - run: npm audit --audit-level=high
```

---

## 4. Métricas

Ver [./metrics.md](./metrics.md)

---

## 5. Decisiones técnicas

**`npm ci` sobre `npm install`**: reproducibilidad garantizada. `npm ci` falla si `package-lock.json` no esta en sync, protegiendo contra instalaciones inconsistentes.

**OWASP Dependency Check en BE**: analiza las dependencias Maven contra la base de datos CVE de NIST. Detecta vulnerabilidades conocidas antes de hacer merge.

**`tsc --noEmit` en FE**: verifica tipos TypeScript sin generar archivos. Permite detectar errores de tipo que no bloquean el build de Vite pero si indican problemas de correctitud.

**Cache de dependencias**: Maven `.m2/repository` y npm `~/.npm` cacheados por hash del archivo de dependencias. Reduce el tiempo de pipeline de ~4 min a ~1.5 min en runs sucesivos.
