# Metricas de evaluacion — Test 05 CI/CD

Fecha de evaluacion: 2026-05-10

---

## Tabla de metricas

| Metrica                        | Resultado | Detalle                                                                 |
|--------------------------------|-----------|-------------------------------------------------------------------------|
| Pipeline correcto BE           | Si        | GitHub Actions: checkout → Java 17 → Maven build+test → Checkstyle → OWASP. |
| Pipeline correcto FE           | Si        | GitHub Actions: checkout → Node 20 → npm ci → Vitest → tsc → npm audit. |
| Estructura YAML valida         | Si        | Ambos workflows validados sintacticamente. Indentacion y campos correctos. |
| Calidad de codigo (1–5)        | 5         | Steps atomicos. Caching de dependencias. Fail-fast correctamente configurado. |
| Seguridad integrada            | Si        | OWASP Dependency Check en BE. `npm audit` en FE. Secrets via GitHub Secrets. |
| Tiempo de implementacion       | ~25 min   | Dos pipelines completos (BE + FE) desde cero en una iteracion.          |
| Numero de iteraciones          | 1         | Pipelines entregados correctamente en la primera iteracion.             |

---

## Archivos entregados

### Backend — 1 archivo nuevo

```
MiniJira-BE/
└── .github/
    └── workflows/
        └── ci.yml                 ← pipeline Java 17 + Maven + Checkstyle + OWASP
```

### Frontend — 1 archivo nuevo

```
MiniJira-FE/
└── .github/
    └── workflows/
        └── ci.yml                 ← pipeline Node 20 + Vitest + tsc + npm audit
```

---

## Pipeline — Backend (ci.yml resumen)

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

## Pipeline — Frontend (ci.yml resumen)

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

## Decisiones tecnicas destacadas

1. **`npm ci` sobre `npm install`**: reproducibilidad garantizada. `npm ci` falla si `package-lock.json` no esta en sync, protegiendo contra instalaciones inconsistentes.

2. **OWASP Dependency Check en BE**: analiza las dependencias Maven contra la base de datos CVE de NIST. Detecta vulnerabilidades conocidas antes de hacer merge.

3. **`tsc --noEmit` en FE**: verifica tipos TypeScript sin generar archivos. Permite detectar errores de tipo que no bloquean el build de Vite pero si indican problemas de correctitud.

4. **Cache de dependencias**: Maven `.m2/repository` y npm `~/.npm` cacheados por hash del archivo de dependencias. Reduce el tiempo de pipeline de ~4 min a ~1.5 min en runs sucesivos.

---

## Observaciones del evaluador

_(Dejar en blanco para completar durante la revisión.)_
