# MiniJira

![branch](https://img.shields.io/badge/branch-development-blue)
![backend](https://img.shields.io/badge/backend-Spring%20Boot%203.2%20%2F%20Java%2017-brightgreen)
![frontend](https://img.shields.io/badge/frontend-React%2018%20%2F%20TypeScript-blue)

Sistema de gestión de tareas estilo Kanban. Permite registrar usuarios, crear proyectos y mover tarjetas entre columnas **TODO / IN_PROGRESS / DONE**. Pensado como base funcional para equipos que necesitan trazabilidad de trabajo sin la complejidad de herramientas enterprise.

---

## Stack tecnológico

| Capa | Tecnología | Versión |
|---|---|---|
| Backend | Spring Boot | 3.2.5 |
| Lenguaje BE | Java | 17 |
| Build BE | Maven | 3.8+ |
| Persistencia | PostgreSQL | 14+ |
| ORM | Spring Data JPA / Hibernate | (incluido en Boot) |
| Seguridad | Spring Security + JWT (jjwt) | 0.12.6 |
| Frontend | React | 18.3 |
| Lenguaje FE | TypeScript | 5.4 |
| Bundler | Vite | 5.3 |
| HTTP client FE | Axios | 1.7 |
| Tests BE | JUnit 5 + H2 (in-memory) | (incluido en Boot) |
| Tests FE | Vitest + Testing Library | (ver package.json) |

---

## Arquitectura

El sistema se compone de tres piezas desplegables de forma independiente:

- **API REST** (Spring Boot) — lógica de negocio, autenticación JWT, acceso a base de datos.
- **SPA** (React + Vite) — interfaz Kanban, consume la API vía Axios.
- **Base de datos** (PostgreSQL) — almacena usuarios, proyectos y tareas.

Para diagramas C4 y decisiones de diseño, ver [docs/architecture.md](docs/architecture.md).

---

## Requisitos previos

| Herramienta | Version minima |
|---|---|
| Java | 17 |
| Maven | 3.8+ |
| PostgreSQL | 14+ |
| Node.js | 18+ |
| npm | 9+ |

---

## Setup completo

### 1. Clonar los repositorios

```bash
git clone -b development https://github.com/Alitocoin/MiniJira-BE.git
git clone -b development https://github.com/Alitocoin/MiniJira-FE.git
```

### 2. Crear la base de datos

```bash
psql -U postgres -c "CREATE DATABASE minijira;"
```

### 3. Variables de entorno del backend

El backend lee su configuración desde variables de entorno. Si la variable no está definida, usa el valor por defecto indicado. Para desarrollo local los defaults funcionan sin cambios; para producción todas deben sobreescribirse explícitamente.

| Variable | Default (desarrollo) | Descripción |
|---|---|---|
| `DB_URL` | `jdbc:postgresql://localhost:5432/minijira` | URL JDBC de PostgreSQL |
| `DB_USERNAME` | `postgres` | Usuario de la base de datos |
| `DB_PASSWORD` | `postgres` | Contrasena de la base de datos |
| `SERVER_PORT` | `8080` | Puerto HTTP del servidor |
| `JPA_DDL_AUTO` | `update` | Estrategia DDL de Hibernate (`update` / `validate` / `none`) |
| `CORS_ALLOWED_ORIGINS` | `http://localhost:5173,http://localhost:3000` | Origenes permitidos (separados por coma) |
| `JWT_SECRET` | `cambiar-esta-clave-en-produccion-minimo-32-chars` | Clave HMAC-SHA para firmar tokens. Minimo 32 caracteres |
| `JWT_EXPIRATION` | `86400000` | Expiracion del token JWT en milisegundos (default = 24 h) |

Exportarlas antes de correr el backend:

```bash
export DB_URL=jdbc:postgresql://localhost:5432/minijira
export DB_USERNAME=postgres
export DB_PASSWORD=postgres
# el resto usa defaults en desarrollo
```

O editarlas directamente en `MiniJira-BE/src/main/resources/application.properties`.

### 4. Ejecutar el backend

```bash
cd MiniJira-BE
mvn spring-boot:run
```

El servidor queda escuchando en `http://localhost:8080`.

### 5. Ejecutar el frontend

```bash
cd MiniJira-FE
npm install
npm run dev
```

La SPA queda disponible en `http://localhost:5173` y apunta automáticamente al backend en `http://localhost:8080/api` (definido en `src/api.ts`).

---

## URLs de acceso

| Servicio | URL |
|---|---|
| Frontend (Vite dev) | http://localhost:5173 |
| Backend API | http://localhost:8080/api |
| H2 Console (solo tests) | http://localhost:8080/h2-console |

---

## Endpoints principales

| Metodo | Ruta | Descripcion | Auth |
|---|---|---|---|
| POST | `/api/auth/register` | Registrar usuario nuevo | No |
| POST | `/api/auth/login` | Iniciar sesion, retorna JWT | No |
| GET | `/api/tasks` | Listar todas las tareas | JWT |
| POST | `/api/tasks` | Crear tarea | JWT |
| PUT | `/api/tasks/{id}` | Actualizar tarea completa | JWT |
| PATCH | `/api/tasks/{id}/status` | Cambiar estado de tarea | JWT |
| DELETE | `/api/tasks/{id}` | Eliminar tarea | JWT |
| GET | `/api/users` | Listar usuarios | JWT |
| POST | `/api/users` | Crear usuario | JWT |
| PUT | `/api/users/{id}` | Actualizar usuario | JWT |
| DELETE | `/api/users/{id}` | Eliminar usuario | JWT |
| GET | `/api/projects` | Listar proyectos | JWT |
| POST | `/api/projects` | Crear proyecto | JWT |
| PUT | `/api/projects/{id}` | Actualizar proyecto | JWT |
| DELETE | `/api/projects/{id}` | Eliminar proyecto | JWT |

Referencia completa con esquemas de request/response: [docs/api-reference.md](docs/api-reference.md) y [docs/openapi.yaml](docs/openapi.yaml).

---

## Testing

**Backend** — usa H2 en memoria, no requiere PostgreSQL corriendo:

```bash
cd MiniJira-BE
mvn test
```

Tests ubicados en `src/test/java/com/minijira/backend/service/`.

**Frontend** — usa Vitest + Testing Library:

```bash
cd MiniJira-FE
npm test
```

Tests ubicados en `src/test/`.

---

## Estructura de carpetas

```
MiniJira-BE/
├── src/
│   ├── main/
│   │   ├── java/com/minijira/backend/
│   │   │   ├── BackendApplication.java       # Entry point Spring Boot
│   │   │   ├── config/
│   │   │   │   ├── CorsConfig.java           # Configuracion CORS
│   │   │   │   └── SecurityConfig.java       # Spring Security + JWT filter
│   │   │   ├── controller/
│   │   │   │   ├── AuthController.java       # POST /api/auth/register|login
│   │   │   │   ├── TaskController.java       # CRUD /api/tasks
│   │   │   │   ├── UserController.java       # CRUD /api/users
│   │   │   │   └── ProjectController.java    # CRUD /api/projects
│   │   │   ├── service/
│   │   │   │   ├── AuthService.java          # Interface
│   │   │   │   ├── AuthServiceImpl.java
│   │   │   │   ├── TaskService.java
│   │   │   │   ├── TaskServiceImpl.java
│   │   │   │   ├── UserService.java
│   │   │   │   ├── UserServiceImpl.java
│   │   │   │   ├── ProjectService.java
│   │   │   │   └── ProjectServiceImpl.java
│   │   │   ├── repository/
│   │   │   │   ├── TaskRepository.java
│   │   │   │   ├── UserRepository.java
│   │   │   │   └── ProjectRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Task.java
│   │   │   │   ├── TaskStatus.java           # Enum: TODO | IN_PROGRESS | DONE
│   │   │   │   ├── User.java
│   │   │   │   └── Project.java
│   │   │   ├── dto/
│   │   │   │   ├── TaskRequest.java / TaskResponse.java
│   │   │   │   ├── UserRequest.java / UserResponse.java
│   │   │   │   ├── ProjectRequest.java / ProjectResponse.java
│   │   │   │   ├── LoginRequest.java / RegisterRequest.java
│   │   │   │   ├── AuthResponse.java
│   │   │   │   ├── StatusUpdateRequest.java
│   │   │   │   └── ErrorResponse.java
│   │   │   ├── exception/
│   │   │   │   ├── GlobalExceptionHandler.java  # @RestControllerAdvice
│   │   │   │   ├── ResourceNotFoundException.java
│   │   │   │   └── BusinessException.java
│   │   │   └── security/
│   │   │       └── JwtUtil.java              # Generacion y validacion de tokens JWT
│   │   └── resources/
│   │       └── application.properties        # Config datasource, JPA, JWT, CORS
│   └── test/
│       └── java/com/minijira/backend/
│           ├── BackendApplicationTests.java
│           └── service/
│               ├── TaskServiceImplTest.java
│               └── UserServiceImplTest.java
└── pom.xml

MiniJira-FE/
├── src/
│   ├── App.tsx                               # Componente raiz, estado global (session, tasks)
│   ├── main.tsx                              # Entry point React
│   ├── api.ts                                # Cliente Axios, todas las llamadas HTTP
│   ├── types.ts                              # Tipos TypeScript (Task, User, Project, Auth)
│   ├── components/
│   │   ├── BoardColumn.tsx                   # Columna Kanban (TODO/IN_PROGRESS/DONE)
│   │   ├── TaskCard.tsx                      # Tarjeta individual de tarea
│   │   ├── CreateTaskForm.tsx                # Formulario de nueva tarea
│   │   ├── LoginForm.tsx                     # Formulario de login
│   │   └── RegisterForm.tsx                  # Formulario de registro
│   └── test/
│       ├── BoardColumn.test.tsx
│       ├── api.test.ts
│       └── setup.ts
├── dist/                                     # Output del build de produccion
├── package.json
└── vite.config.ts
```

---

## Documentacion tecnica

| Documento | Contenido |
|---|---|
| [docs/architecture.md](docs/architecture.md) | Diagramas C4 y decisiones de diseno |
| [docs/api-reference.md](docs/api-reference.md) | Referencia completa de la API REST |
| [docs/openapi.yaml](docs/openapi.yaml) | Especificacion OpenAPI 3.0 |
| [docs/c4-l4-code.md](docs/c4-l4-code.md) | Diagramas de codigo (nivel L4) |
| [docs/data-dictionary.md](docs/data-dictionary.md) | Diccionario de datos del modelo |
| [docs/developer-guide.md](docs/developer-guide.md) | Guia para desarrolladores nuevos |
| [docs/deployment-guide.md](docs/deployment-guide.md) | Guia de despliegue a produccion |
