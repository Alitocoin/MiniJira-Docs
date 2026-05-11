# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

Formato: [Keep a Changelog](https://keepachangelog.com/es/1.0.0/)
Versionado: [SemVer](https://semver.org/lang/es/)

---

## [0.5.0] — 2026-05-11

### Added

- **Sistema MiniJira completo** — API REST con Spring Boot 3 / Java 17 y tablero Kanban con React 18 / TypeScript / Vite. Base de datos PostgreSQL.
- **Autenticación JWT stateless** — endpoints `POST /api/auth/register` y `POST /api/auth/login`. Passwords hasheados con BCrypt (factor 10). Token firmado con HMAC-SHA, clave configurable via variable de entorno `JWT_SECRET`. Interceptor axios en el frontend inyecta el Bearer token en cada petición.
- **CRUD completo de Tasks** — 7 endpoints REST: listar, obtener por id, crear, actualizar, actualizar status (PATCH), eliminar, y listar por proyecto. Status permitidos: `TODO`, `IN_PROGRESS`, `DONE`.
- **CRUD completo de Users** — 5 endpoints REST: listar, obtener por id, crear, actualizar, eliminar. Validación de unicidad de username y email.
- **CRUD completo de Projects** — 3 endpoints REST: listar, obtener por id, crear. Total del sistema: 15 endpoints REST.
- **Tablero Kanban** — tres columnas (Por Hacer, En Progreso, Hecho) con conteo de tareas por columna, navegación entre estados con botones de avance, estado vacío por columna, y eliminación de tareas.
- **Formularios de auth en frontend** — `LoginForm.tsx` y `RegisterForm.tsx` con manejo de estado de sesión en `App.tsx`.
- **24 unit tests backend** (JUnit 5 + Mockito): `TaskServiceImplTest` (15 tests) y `UserServiceImplTest` (12 tests). Cubren happy path, `ResourceNotFoundException` y `BusinessException`. Pendientes de ejecución con JVM.
- **21 unit tests frontend** (Vitest + Testing Library): `BoardColumn.test.tsx` (10 tests) y `api.test.ts` (11 tests). Estado: 21/21 verdes.
- **Pipeline CI/CD GitHub Actions** en BE y FE con tres jobs: `build-and-test` (compilación + tests), `lint` (Checkstyle en BE / `tsc --noEmit` en FE), `security` (OWASP Dependency-Check en BE / `npm audit` en FE). Jobs paralelos con artefactos publicados.
- **Documentación técnica completa** en `MiniJira-Docs`:
  - C4 Model L1 (Context), L2 (Container), L3 (Component — API), L4 (Code — secuencia JWT)
  - OpenAPI 3.0 con los 15 endpoints documentados
  - Data Dictionary con todas las entidades y sus campos
  - README ejecutivo con setup local en 3 pasos
  - Developer Guide con convenciones de código y flujo de branches
  - Deployment Guide con instrucciones para Cloud Run

### Fixed

- `handleStatusChange` en `App.tsx` no limpiaba `setError(null)` antes del PATCH. Un error previo mostrado en la UI permanecía visible aunque el backend procesara el cambio exitosamente. Fix: una línea `setError(null)` al inicio del handler, consistente con el patrón de `fetchTasks`.

### Security

- Passwords almacenados exclusivamente como hash BCrypt con factor de costo 10. Nunca en texto plano.
- Token JWT firmado con HMAC-SHA; la clave secreta se inyecta via variable de entorno `JWT_SECRET` — no hay valores hardcodeados en `application.properties`.
- Spring Security configurado como stateless (sin sesión de servidor). La ruta `/api/auth/**` es pública; el resto requiere token válido (pendiente activación antes de merge a `main` — ver R04 en QA Test Plan).

---

[0.5.0]: https://github.com/Alitocoin/MiniJira-Docs/compare/main...improvement/docs-completa
