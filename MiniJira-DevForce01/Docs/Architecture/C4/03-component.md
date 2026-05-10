# C4 Nivel 3 — Component

Zoom dentro del contenedor **API REST** (Spring Boot). Muestra cómo se organiza internamente el backend.

## Diagrama

```mermaid
C4Component
  title Diagrama de componentes — API REST (Spring Boot 3.2)

  Container_Ext(spa, "SPA", "React 18 + Vite", "Interfaz Kanban")
  ContainerDb_Ext(db, "PostgreSQL", "Base de datos relacional")

  Container_Boundary(api, "API REST - Spring Boot 3.2 / Java 17") {

    Component(tc, "TaskController", "Spring @RestController", "Expone 7 endpoints REST para /api/tasks. Delega lógica a TaskService.")
    Component(uc, "UserController", "Spring @RestController", "Expone 3 endpoints REST para /api/users. Delega lógica a UserService.")
    Component(pc, "ProjectController", "Spring @RestController", "Expone 3 endpoints REST para /api/projects. Delega lógica a ProjectService.")

    Component(ts, "TaskService", "Spring @Service", "Lógica de negocio de tareas: CRUD completo, filtrado por status, resolución de relaciones User/Project.")
    Component(us, "UserService", "Spring @Service", "Lógica de negocio de usuarios: CRUD, validación de email único.")
    Component(ps, "ProjectService", "Spring @Service", "Lógica de negocio de proyectos: CRUD básico.")

    Component(tr, "TaskRepository", "Spring Data JPA", "Acceso a tabla tasks. JPQL custom: findAllWithRelations, findByStatusWithRelations.")
    Component(ur, "UserRepository", "Spring Data JPA", "Acceso a tabla users. Métodos: findByEmail, existsByEmail.")
    Component(pr, "ProjectRepository", "Spring Data JPA", "Acceso a tabla projects. Hereda JpaRepository.")

    Component(dto, "DTOs (Request/Response)", "POJO + Lombok", "TaskRequest, TaskStatusRequest, UserRequest, ProjectRequest. TaskResponse, UserResponse, ProjectResponse.")
    Component(model, "Modelos JPA", "Jakarta Persistence", "Task, User, Project, TaskStatus (enum: TODO / IN_PROGRESS / DONE).")
    Component(geh, "GlobalExceptionHandler", "Spring @RestControllerAdvice", "Captura ResourceNotFoundException (404), BusinessException (409), MethodArgumentNotValidException (400) y Exception genérica (500).")
    Component(cors, "CorsConfig", "Spring @Configuration", "Habilita CORS en /api/**. Permite todos los orígenes. Configurable via env cors.allowed-origins.")
  }

  Rel(spa, tc, "HTTP/JSON")
  Rel(spa, uc, "HTTP/JSON")
  Rel(spa, pc, "HTTP/JSON")

  Rel(tc, ts, "llama")
  Rel(uc, us, "llama")
  Rel(pc, ps, "llama")

  Rel(ts, tr, "usa")
  Rel(ts, ur, "usa")
  Rel(ts, pr, "usa")
  Rel(us, ur, "usa")
  Rel(ps, pr, "usa")

  Rel(tr, db, "JDBC")
  Rel(ur, db, "JDBC")
  Rel(pr, db, "JDBC")

  Rel(tc, dto, "recibe/devuelve")
  Rel(uc, dto, "recibe/devuelve")
  Rel(pc, dto, "recibe/devuelve")

  Rel(tr, model, "mapea")
  Rel(ur, model, "mapea")
  Rel(pr, model, "mapea")
```

## Descripción de componentes

### Capa Controller

Recibe requests HTTP, valida con `@Valid`, delega al service correspondiente. No tiene lógica de negocio propia.

| Componente | Endpoints | Responsabilidad |
|------------|-----------|-----------------|
| `TaskController` | 7 — `/api/tasks` | CRUD completo de tareas + filtrado por status + actualización parcial de status |
| `UserController` | 3 — `/api/users` | Listar, obtener por ID, crear usuario |
| `ProjectController` | 3 — `/api/projects` | Listar, obtener por ID, crear proyecto |

### Capa Service

Contiene la lógica de negocio. Todas las clases son `@Transactional(readOnly = true)` por defecto; los métodos de escritura tienen `@Transactional` individual.

| Componente | Responsabilidad clave |
|------------|-----------------------|
| `TaskService` | Resuelve las relaciones User y Project antes de persistir. Mapea entidades a `TaskResponse` con builder. |
| `UserService` | Valida unicidad de email con `existsByEmail` antes de crear. Lanza `BusinessException` en duplicados. |
| `ProjectService` | CRUD simple, sin reglas de negocio adicionales en v1. |

### Capa Repository

Todas extienden `JpaRepository<T, Long>`. `TaskRepository` agrega dos queries JPQL con `LEFT JOIN FETCH` para evitar el problema N+1 al cargar las relaciones `assignedUser` y `project`.

### Modelos JPA

| Entidad | Tabla | Relaciones |
|---------|-------|------------|
| `Task` | `tasks` | `@ManyToOne` → `User` (LAZY), `@ManyToOne` → `Project` (LAZY) |
| `User` | `users` | — |
| `Project` | `projects` | — |
| `TaskStatus` | — (enum) | Valores: `TODO`, `IN_PROGRESS`, `DONE` |

### DTOs

Separan el contrato HTTP del modelo de persistencia. Los `Request` tienen anotaciones de validación Jakarta (`@NotBlank`, `@NotNull`, `@Email`). Los `Response` usan el patrón Builder de Lombok.

### GlobalExceptionHandler

Centraliza el manejo de errores. Devuelve `ErrorResponse` con campos: `status`, `error`, `message`, `path`, `timestamp`, `details`.

| Excepción capturada | HTTP status | Caso de uso |
|--------------------|-------------|-------------|
| `ResourceNotFoundException` | 404 | Entidad no encontrada por ID |
| `BusinessException` | 409 | Email de usuario duplicado |
| `MethodArgumentNotValidException` | 400 | Fallo de validación Jakarta en request |
| `Exception` (genérica) | 500 | Error inesperado |

---

Nivel anterior: [02-container.md](02-container.md)
