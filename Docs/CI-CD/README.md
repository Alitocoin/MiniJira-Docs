# Documentación CI/CD — MiniJira

Indice de documentación del sistema de integración continua del proyecto MiniJira.

## Repositorios involucrados

| Repo | Branch de trabajo |
|---|---|
| `MiniJira-FE` | `test-05-cicd_v01` |
| `MiniJira-BE` | `test-05-cicd_v01` |
| `MiniJira-Docs` | `test-05-cicd_v01` |

## Propósito del sistema CI/CD

Cada push o pull request a `test-05-cicd_v01` dispara pipelines automáticos en GitHub Actions que garantizan:

1. Las dependencias instalan sin errores.
2. El código pasa lint y tests unitarios.
3. El build de producción compila exitosamente.
4. No existen vulnerabilidades conocidas de severidad alta o crítica.

Los pipelines fallan rápido: si un paso bloqueante falla, los siguientes no se ejecutan. Los artefactos (build compilado, reporte OWASP) se suben incluso cuando hay fallos, para facilitar el diagnóstico.

## Archivos en este directorio

| Archivo | Contenido |
|---|---|
| `frontend-pipeline.md` | Documentación técnica del pipeline GitHub Actions para MiniJira-FE |
| `backend-pipeline.md` | Documentación técnica del pipeline GitHub Actions para MiniJira-BE |
| `local-ci-execution.md` | Cómo ejecutar el CI localmente antes de hacer push |
| `security-scan.md` | Cómo funcionan npm audit y OWASP dependency-check |
| `troubleshooting.md` | Soluciones a problemas comunes en los pipelines |
| `execution-report.md` | Reporte de implementación del sprint CI/CD (Test-05) |

## Estado actual de los pipelines

**Frontend** — todos los componentes activos:
- Lint: `eslint.config.js` presente, pipeline corre `npm run lint`
- Tests: archivos Vitest en `src/__tests__/`, pipeline corre `npm run test -- --run`
- Build: `npm run build` — falla el pipeline si falla
- Audit: `package-lock.json` presente, corre `npm audit --audit-level=high`

**Backend** — todos los componentes activos:
- Tests: JUnit + Mockito en `src/test/`, pipeline corre `mvn clean test`
- Verify: `mvn verify` — falla el pipeline si falla
- OWASP: dependency-check configurado con `failBuildOnCVSS=7`

## Archivos de workflow

```
MiniJira-FE/.github/workflows/ci-frontend.yml
MiniJira-BE/.github/workflows/ci-backend.yml
```
