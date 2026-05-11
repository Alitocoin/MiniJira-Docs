# ADR-005 — Estrategia de expiración y refresh de tokens JWT

**Fecha:** 2026-05-11
**Estado:** Proposed
**Autores:** Jose Perez, Luis Barrios
**Relacionado con:** ADR-002 (autenticación JWT stateless), Test 03 (Auth JWT)

---

## Contexto

MiniJira implementa autenticación JWT stateless (ver ADR-002). En la configuración actual:

- El token se firma con HMAC-SHA256 (HS256) usando `jjwt 0.12.6`.
- La expiración es de **24 horas**, configurable via `JWT_EXPIRATION` en variables de entorno.
- El token se almacena en **`localStorage`** del browser (decisión de implementación del frontend).
- No existe mecanismo de refresh token.
- No existe blacklist de tokens invalidados.

El ADR-002 registró como deuda técnica: *"Evaluar expiración de token y mecanismo de refresh si el proyecto escala"*. Esta evaluación se hace ahora porque el equipo está considerando mover el proyecto de demo/MVP a un entorno de producción con múltiples usuarios reales.

Los riesgos concretos que motivan esta decisión son:

1. **Ventana de robo larga**: si un token de 24 h es robado (XSS, red insegura), el atacante tiene acceso durante todo ese periodo. No hay forma de invalidarlo sin reiniciar el servidor o cambiar el secreto global (lo que invalida todos los tokens).
2. **`localStorage` como vector XSS**: cualquier script inyectado puede leer el token. Esta es la segunda parte del problema de superficie de ataque.
3. **Sin logout efectivo**: el backend no tiene endpoint de logout que invalide el token; el frontend solo borra el estado en memoria.

---

## Alternativas consideradas

### Opcion A — Status quo: token de larga duracion (24 h), sin refresh

**Descripcion:** Mantener la configuracion actual. El token dura 24 horas y no hay mecanismo de renovacion. El usuario se autentica una vez por dia.

**Ventajas:**
- Cero cambios de infraestructura o codigo.
- UX simple: el usuario no es interrumpido por expiraciones durante su jornada laboral.
- Sin dependencia adicional en base de datos para refresh tokens.

**Desventajas / riesgos:**
- Ventana de ataque de 24 horas si el token es comprometido. No mitigable sin reiniciar el secreto global (invalida todos los usuarios simultáneamente).
- `localStorage` con token de larga duracion es una combinacion de alto riesgo segun OWASP Top 10 (A07:2021 Identification and Authentication Failures).
- Inaceptable para produccion con datos sensibles reales.
- No cumple con buenas practicas de seguridad para aplicaciones bancarias o con datos personales.

---

### Opcion B — Token de corta duracion (15 min) + refresh token en base de datos

**Descripcion:** El access token expira en 15 minutos. Al login se emite ademas un refresh token de larga duracion (ej. 7 dias) que se almacena en la tabla `refresh_tokens` en PostgreSQL. Cuando el access token expira, el frontend llama a `POST /api/auth/refresh` enviando el refresh token para obtener un nuevo access token.

**Ventajas:**
- Ventana de ataque reducida a 15 minutos para el access token.
- El refresh token puede ser invalidado individualmente en la base de datos (logout efectivo, revocacion por usuario comprometido).
- Permite implementar "recordar sesion" con refresh tokens de duracion configurable.

**Desventajas / riesgos:**
- Requiere nueva tabla en PostgreSQL (`refresh_tokens`: id, user_id, token_hash, expires_at, revoked).
- El refresh token sigue expuesto en `localStorage` si se almacena ahi (mismo vector XSS).
- Agrega una peticion de red cada 15 minutos por usuario activo (carga en `/api/auth/refresh`).
- Mayor complejidad en el frontend: logica de interceptor axios para detectar 401 y reintentar con refresh.
- Si el refresh token tambien esta en `localStorage`, se gana poco en seguridad frente a XSS.

---

### Opcion C — Token de corta duracion (15 min) + refresh token en cookie httpOnly (sliding window)

**Descripcion:** El access token expira en 15 minutos (puede seguir en `localStorage` o en memoria). El refresh token se almacena en una **cookie `httpOnly; Secure; SameSite=Strict`**, invisible para JavaScript. El endpoint `POST /api/auth/refresh` lee la cookie automaticamente. Con sliding window, cada refresh exitoso extiende la vida de la sesion.

**Ventajas:**
- El refresh token es **inaccesible para JavaScript** (mitiga XSS de forma estructural).
- Revocacion individual posible si se persiste el token o su hash en base de datos.
- Mejor postura de seguridad general: combina corta duracion del access token con proteccion del refresh token contra XSS.
- Sliding window da una UX fluida: la sesion se extiende mientras el usuario esta activo.

**Desventajas / riesgos:**
- Requiere configuracion CORS cuidadosa (`credentials: true` en axios, `allowedOrigins` explicito — no wildcard `*`).
- Introduce vulnerabilidad CSRF en el endpoint `/api/auth/refresh` al usar cookies. Mitigable con `SameSite=Strict` + token CSRF si el frontend esta en diferente origen.
- Mayor complejidad de configuracion en Spring Boot (`CookieCsrfTokenRepository` o similar).
- En despliegues Cloud Run con dominios distintos para FE y BE, el atributo `SameSite` puede complicar el flujo (requiere dominio compartido o configuracion especifica).
- Mas partes moviles: si el equipo no tiene experiencia con el modelo cookie+JWT, el riesgo de configuracion incorrecta es real.

---

### Opcion D — Re-login obligatorio al expirar (UX simple, sin refresh)

**Descripcion:** Reducir la expiracion del token a un valor mas corto (ej. 2-4 horas) y forzar re-login cuando el token expire. Sin refresh token de ningun tipo. El frontend detecta el 401 y redirige al login.

**Ventajas:**
- Ventana de ataque reducida sin agregar complejidad de refresh.
- Sin nueva infraestructura (no tabla adicional, no cookies nuevas).
- Logica de frontend simple: interceptor 401 → redirigir a `/login`.
- Apropiado para herramientas internas donde el re-login cada pocas horas no es un problema de UX.

**Desventajas / riesgos:**
- UX disruptiva: el usuario pierde el contexto de trabajo al ser redirigido al login sin previo aviso.
- El token sigue en `localStorage`; si la ventana se reduce a 2-4 h, el riesgo XSS sigue siendo real pero acotado.
- No permite "recordar sesion" ni flujos de larga duracion (ej. usuario que deja el tablero abierto toda la noche).
- Sin logout efectivo: el token sigue valido hasta que expire naturalmente.

---

## Decision

**Para la etapa actual (demo/MVP):** se recomienda mantener la **Opcion A** con la condicion de que el equipo acepte formalmente el riesgo de seguridad y el proyecto no maneje datos personales reales ni credenciales bancarias de usuarios finales.

**Para una eventual transicion a produccion con usuarios reales:** se recomienda implementar la **Opcion C** (token de corta duracion + refresh token en cookie httpOnly). Es la opcion con mejor balance seguridad/UX, aunque requiere planificacion cuidadosa del deploy (CORS, dominios, CSRF).

La **Opcion D** es una alternativa valida como paso intermedio si el equipo quiere mejorar la seguridad sin agregar complejidad de refresh, especialmente para usuarios internos que toleran re-login periodico.

La **Opcion B** no se recomienda porque almacenar el refresh token en `localStorage` (como lo hace el frontend actualmente) anula la principal ventaja frente al status quo.

**Esta decision no compromete implementacion.** Cualquier cambio requiere una task aprobada en el backlog con criterios de aceptacion definidos.

---

## Consecuencias

### Positivas

- El equipo tiene documentado formalmente el riesgo de la configuracion actual.
- Si se decide avanzar a produccion, existe una hoja de ruta clara (Opcion C) con los trade-offs conocidos.
- La evaluacion reduce la ambiguedad tecnica en futuras discusiones de arquitectura.

### Negativas / compromisos aceptados

- Mientras se permanezca en Opcion A, el riesgo de token comprometido con ventana de 24 h es aceptado conscientemente.
- El token en `localStorage` sigue siendo un vector XSS activo; se acepta porque el proyecto es demo y no maneja datos sensibles reales.
- Si el equipo elige Opcion C en el futuro, hay un esfuerzo de refactorizacion no trivial tanto en backend (Spring Boot Security, cookie management) como en frontend (axios credentials, manejo de CSRF).

---

## Criterios de implementacion (si se elige avanzar con Opcion C)

- [ ] Reducir `JWT_EXPIRATION` a 15 minutos (900000 ms) en variables de entorno de produccion.
- [ ] Crear entidad y repositorio `RefreshToken` en PostgreSQL con campos: `id`, `userEmail`, `tokenHash`, `expiresAt`, `revoked`.
- [ ] Implementar `POST /api/auth/refresh` que lea la cookie `refresh_token` (httpOnly), valide contra la DB y emita nuevo access token.
- [ ] Configurar Spring Boot para emitir cookie `httpOnly; Secure; SameSite=Strict; Path=/api/auth/refresh` en el response de login.
- [ ] Actualizar `SecurityConfig` para permitir `/api/auth/refresh` sin Bearer token pero con validacion de cookie.
- [ ] Actualizar interceptor axios en el frontend: detectar 401, llamar a `/api/auth/refresh` con `credentials: true`, reintentar la peticion original.
- [ ] Verificar configuracion CORS: `allowCredentials(true)` + origen explicito (no `*`).
- [ ] Escribir tests de integracion para el flujo completo: login → acceso → expiracion → refresh → acceso renovado.
- [ ] Revisar configuracion de dominios en Cloud Run para garantizar compatibilidad con `SameSite=Strict`.
