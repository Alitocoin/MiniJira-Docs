# Test 03 — Feature de Autenticación JWT (Backend + Frontend)

**Fecha:** 2026-05-10  
**Branch:** `feat/test-03-user-feature`  
**Estado:** Completado

---

## 1. Descripción de la feature

Implementación del módulo completo de autenticación stateless usando JWT (JSON Web Tokens) en el backend Spring Boot y la integración correspondiente en el frontend React.

**Objetivo:** Proteger todos los endpoints de la API con autenticación Bearer token. Los usuarios deben registrarse o iniciar sesión para obtener un JWT y adjuntarlo en cada request subsiguiente.

---

## 2. Arquitectura del módulo auth

### Backend (Spring Boot)

```
com.minijira.backend/
├── controller/
│   └── AuthController.java       ← POST /api/auth/login, POST /api/auth/register
├── service/
│   ├── AuthService.java          ← interfaz
│   └── AuthServiceImpl.java      ← BCrypt + JWT generation
├── security/
│   ├── JwtUtil.java              ← HMAC-SHA256 sign/verify (jjwt 0.12.6)
│   └── SecurityConfig.java       ← Spring Security stateless, rutas públicas
└── dto/
    ├── AuthResponse.java         ← { token, type, username, email }
    ├── LoginRequest.java         ← { email, password }
    └── RegisterRequest.java      ← { username, email, password }
```

### Frontend (React + TypeScript)

```
src/
├── components/
│   ├── LoginForm.tsx             ← form con email + password, POST /api/auth/login
│   └── RegisterForm.tsx          ← form con username + email + password
├── api.ts                        ← login(), register(), interceptor Bearer token
└── types.ts                      ← AuthResponse, AuthSession
```

---

## 3. Archivos modificados / creados

### Backend

| Archivo | Tipo |
|---------|------|
| `AuthController.java` | Nuevo |
| `AuthService.java` | Nuevo |
| `AuthServiceImpl.java` | Nuevo |
| `JwtUtil.java` | Nuevo |
| `SecurityConfig.java` | Nuevo |
| `AuthResponse.java` | Nuevo |
| `LoginRequest.java` | Nuevo |
| `RegisterRequest.java` | Nuevo |
| `User.java` | Modificado — campo `password` + getter/setter |
| `UserRepository.java` | Modificado — `findByEmail()` |
| `application.properties` | Modificado — `jwt.secret`, `jwt.expiration-ms` |
| `pom.xml` | Modificado — `spring-boot-starter-security`, `jjwt` 0.12.6 |

### Frontend

| Archivo | Tipo |
|---------|------|
| `LoginForm.tsx` | Nuevo |
| `RegisterForm.tsx` | Nuevo |
| `App.tsx` | Modificado — session management, routing auth/board |
| `api.ts` | Modificado — `login()`, `register()`, interceptor |
| `types.ts` | Modificado — `AuthResponse`, `AuthSession` |

---

## 4. Flujo de autenticación

```
Cliente → POST /api/auth/login { email, password }
         ← 200 { token: "eyJ...", type: "Bearer", username, email }

Cliente → GET /api/tasks
         Authorization: Bearer eyJ...
         ← 200 [...]

Cliente → GET /api/tasks (sin token)
         ← 401 Unauthorized
```

---

## 5. Configuración de seguridad

- **Rutas públicas**: `POST /api/auth/login`, `POST /api/auth/register`
- **Rutas protegidas**: todos los demás endpoints (`/api/tasks/**`, `/api/users/**`, `/api/projects/**`)
- **Algoritmo JWT**: HMAC-SHA256 (HS256)
- **Expiración**: 24 horas (`86400000 ms`) configurable via `JWT_EXPIRATION`
- **Hash de contraseñas**: BCrypt con factor de costo por defecto

---

## 6. Métricas

Ver [`docs/test-03-metrics.md`](./test-03-metrics.md)

---

## 7. Decisiones técnicas

**JWT stateless**: se optó por JWT sin sesiones en servidor. Cada token es auto-contenido y contiene el subject (email). Elimina la necesidad de almacenamiento de sesiones y escala horizontalmente sin sticky sessions.

**BCrypt para passwords**: algoritmo de hashing adaptativo. El factor de costo por defecto de Spring Security (`10`) provee resistencia adecuada a ataques de fuerza bruta en el contexto de una app demo.

**Interceptor axios**: el token JWT se almacena en `localStorage` y se adjunta automáticamente a cada request via interceptor, evitando pasar el token manualmente en cada llamada a la API.

**SecurityConfig stateless**: Spring Security configurado con `SessionCreationPolicy.STATELESS`. No se crean cookies de sesión, lo que es consistente con el enfoque API REST + SPA.
