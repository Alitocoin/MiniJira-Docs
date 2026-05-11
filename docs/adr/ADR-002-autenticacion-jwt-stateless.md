# ADR-002 — Autenticación JWT stateless con Spring Security

**Estado:** Aceptado
**Fecha:** 2026-05-11
**Autores:** Carlos Frost (Backend), Paula Aguilar (Frontend), Luis Barrios (Docs)
**Relacionado con:** Test 03 (Auth JWT) — branches `feat/test-03-user-feature`

---

## Contexto

MiniJira necesitaba un mecanismo de autenticación para los endpoints de la API. Las opciones evaluadas fueron:

1. **Sesión en servidor (stateful)** — Spring Security con `HttpSession`. Escala mal horizontalmente y requiere almacenamiento compartido (Redis u otro) en entornos multi-instancia.
2. **JWT stateless** — token firmado emitido en login, enviado en cada petición como `Authorization: Bearer <token>`. Sin estado en servidor.
3. **OAuth2 / OpenID Connect** — delegar a un proveedor externo (Google, Auth0). Excede el alcance del proyecto.

El sistema no requiere invalidación de tokens en el corto plazo y el despliegue objetivo es Cloud Run (donde las instancias pueden escalar horizontalmente).

---

## Decisión

Se implementó **JWT stateless con Spring Security** bajo las siguientes condiciones:

- **Emisión** — `POST /api/auth/register` y `POST /api/auth/login` son los únicos endpoints públicos. En login exitoso se retorna `{ "token": "<jwt>" }`.
- **Firma** — HMAC-SHA. La clave secreta se inyecta via variable de entorno `JWT_SECRET`. Sin valores hardcodeados en `application.properties`.
- **Passwords** — BCrypt con factor de costo 10. Nunca almacenados en texto plano.
- **Configuración de Spring Security** — `SessionCreationPolicy.STATELESS`. La cadena de filtros incluye un `JwtAuthenticationFilter` que extrae y valida el token en cada petición.
- **Endpoints protegidos** — la decisión intencional para el Test 03 fue dejar el CRUD (`/api/tasks/**`, `/api/users/**`, `/api/projects/**`) **sin protección activa** para no bloquear el frontend mientras se estabilizaba el flujo. Se documentó como riesgo R04 y debe resolverse antes de merge a `main`.
- **Frontend** — interceptor axios configurado para inyectar el header `Authorization: Bearer <token>` en todas las peticiones. Token almacenado en estado de `App.tsx` (no en `localStorage`, evitando XSS de persistencia).

### Archivos involucrados

```
BE: AuthController.java, AuthServiceImpl.java, JwtUtil.java,
    SecurityConfig.java, JwtAuthenticationFilter.java,
    RegisterRequest.java, LoginRequest.java, AuthResponse.java

FE: LoginForm.tsx, RegisterForm.tsx, api.ts (interceptor), App.tsx
```

---

## Consecuencias

**Positivas:**
- Escala horizontalmente sin estado compartido entre instancias de Cloud Run.
- El frontend puede correr en un dominio distinto al backend sin necesidad de cookies (evita problemas de SameSite/CORS con sesiones).
- Implementación autocontenida: no depende de servicios externos de autenticación.

**Negativas:**
- La invalidación de tokens antes de su expiración requiere una blacklist (Redis u otra solución). No implementada. Si un usuario cierra sesión, el token sigue siendo válido hasta que expire.
- Los tests unitarios del flujo de auth (`AuthServiceImpl`) no están escritos todavía. Ver gap en QA Test Plan.
- `AuthController` no tiene tests de integración; el contrato se verificó manualmente.

**Deuda técnica registrada:**
- R04 (QA Test Plan): activar protección JWT en endpoints CRUD antes de merge a `main`.
- Implementar tests unitarios para `AuthServiceImpl`.
- Evaluar expiración de token y mecanismo de refresh si el proyecto escala.
