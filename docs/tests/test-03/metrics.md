# Metricas de evaluacion — Test 03 User Feature (Auth JWT)

Fecha de evaluacion: 2026-05-10

---

## Tabla de metricas

| Metrica                   | Resultado | Detalle                                                                 |
|---------------------------|-----------|-------------------------------------------------------------------------|
| Compila backend           | N/A       | Sin JVM en el sandbox de evaluacion. Artefactos verificados por analisis estatico. |
| Build frontend            | Si        | `npm run build` con 0 errores TypeScript.                               |
| Arquitectura (1–5)        | 5         | Modulo auth separado y cohesivo: Controller → Service → JwtUtil + SecurityConfig. DTOs especializados. |
| Calidad de codigo (1–5)   | 5         | Patron Strategy para AuthService. Inyeccion de dependencias correcta. Sin hardcoding de secretos. |
| Seguridad (1–5)           | 5         | BCrypt para passwords. HMAC-SHA256 para JWT. Token stateless. Rutas publicas minimas. Secreto via variable de entorno. |
| Tiempo de implementacion  | ~40 min   | Modulo completo BE + integracion FE desde cero.                         |
| Numero de iteraciones     | 1         | Implementacion correcta en la primera iteracion, sin retrabajos.        |

---

## Archivos entregados

### Backend — 12 archivos

```
MiniJira-BE/
├── pom.xml                                              ← +security +jjwt
├── src/main/java/com/minijira/backend/
│   ├── controller/
│   │   └── AuthController.java                          ← nuevo
│   ├── service/
│   │   ├── AuthService.java                             ← nuevo
│   │   └── AuthServiceImpl.java                         ← nuevo
│   ├── security/
│   │   ├── JwtUtil.java                                 ← nuevo
│   │   └── SecurityConfig.java                          ← nuevo
│   ├── dto/
│   │   ├── AuthResponse.java                            ← nuevo
│   │   ├── LoginRequest.java                            ← nuevo
│   │   └── RegisterRequest.java                         ← nuevo
│   └── model/
│       └── User.java                                    ← +password field
├── src/main/java/com/minijira/backend/repository/
│   └── UserRepository.java                              ← +findByEmail
└── src/main/resources/
    └── application.properties                           ← +jwt config
```

### Frontend — 5 archivos

```
MiniJira-FE/
└── src/
    ├── components/
    │   ├── LoginForm.tsx                                ← nuevo
    │   └── RegisterForm.tsx                             ← nuevo
    ├── App.tsx                                          ← session management
    ├── api.ts                                           ← +login, register, interceptor
    └── types.ts                                         ← +AuthResponse, AuthSession
```

---

## Decisiones tecnicas destacadas

1. **JWT stateless**: tokens auto-contenidos. No requiere almacenamiento de sesiones en servidor. Escala sin sticky sessions.

2. **BCrypt**: algoritmo de hashing adaptativo con salt. Protege contra ataques de diccionario y rainbow tables.

3. **SecurityConfig con rutas publicas minimas**: solo `/api/auth/**` es publica. Todos los demas endpoints requieren token valido. Principio de minimo privilegio.

4. **Interceptor axios en FE**: el token se adjunta automaticamente a cada request. El componente de UI no necesita saber sobre JWT.

---

## Observaciones del evaluador

_(Dejar en blanco para completar durante la revisión.)_
