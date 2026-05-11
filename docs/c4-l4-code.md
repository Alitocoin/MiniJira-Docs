# C4 Level 4 — Diagramas de clases

Nivel de código del modelo C4. Documenta las clases, interfaces, enums y módulos reales
del sistema MiniJira. Generado a partir del código fuente de los repositorios
`MiniJira-BE` (branch `development`) y `MiniJira-FE` (branch `development`).

---

## Backend — Entidades JPA

Las entidades mapean directamente las tablas de PostgreSQL. Todas usan `GenerationType.IDENTITY`
para sus PKs y registran `createdAt` via `@PrePersist`.

```mermaid
classDiagram
    class User {
        +Long id
        +String username
        +String email
        +LocalDateTime createdAt
        +prePersist() void
    }

    class Project {
        +Long id
        +String name
        +String description
        +LocalDateTime createdAt
        +prePersist() void
    }

    class Task {
        +Long id
        +String title
        +String description
        +TaskStatus status
        +Integer storyPoints
        +Double estimatedHours
        +LocalDate startDate
        +LocalDate endDate
        +LocalDateTime createdAt
        +User assignee
        +Project project
        +prePersist() void
    }

    class TaskStatus {
        <<enumeration>>
        TODO
        IN_PROGRESS
        DONE
    }

    Task --> TaskStatus : status
    Task "many" --> "0..1" User : assignee (LAZY)
    Task "many" --> "0..1" Project : project (LAZY)
```

---

## Backend — Repositorios

Extienden `JpaRepository<T, Long>`. Los métodos derivados son los únicos métodos
personalizados presentes en el código; el resto viene de la interfaz base de Spring Data.

```mermaid
classDiagram
    class JpaRepository~T_ID~ {
        <<interface>>
        +findAll() List~T~
        +findById(ID) Optional~T~
        +save(T) T
        +deleteById(ID) void
        +existsById(ID) boolean
    }

    class TaskRepository {
        <<interface>>
        +findByStatus(TaskStatus) List~Task~
        +findByProjectId(Long) List~Task~
        +findByAssigneeId(Long) List~Task~
    }

    class UserRepository {
        <<interface>>
        +existsByUsername(String) boolean
        +existsByEmail(String) boolean
    }

    class ProjectRepository {
        <<interface>>
    }

    TaskRepository --|> JpaRepository : extends
    UserRepository --|> JpaRepository : extends
    ProjectRepository --|> JpaRepository : extends

    TaskRepository ..> Task : manages
    UserRepository ..> User : manages
    ProjectRepository ..> Project : manages
```

---

## Backend — Servicios

Cada dominio tiene una interfaz y su implementación. `TaskServiceImpl` es el mas complejo:
depende de los tres repositorios para resolver referencias de `assignee` y `project`.

```mermaid
classDiagram
    class TaskService {
        <<interface>>
        +findAll() List~TaskResponse~
        +findById(Long) TaskResponse
        +create(TaskRequest) TaskResponse
        +update(Long, TaskRequest) TaskResponse
        +updateStatus(Long, StatusUpdateRequest) TaskResponse
        +delete(Long) void
    }

    class TaskServiceImpl {
        -TaskRepository taskRepository
        -UserRepository userRepository
        -ProjectRepository projectRepository
        +findAll() List~TaskResponse~
        +findById(Long) TaskResponse
        +create(TaskRequest) TaskResponse
        +update(Long, TaskRequest) TaskResponse
        +updateStatus(Long, StatusUpdateRequest) TaskResponse
        +delete(Long) void
        -applyRequestToTask(TaskRequest, Task) void
        -findTaskOrThrow(Long) Task
        -toResponse(Task) TaskResponse
    }

    class UserService {
        <<interface>>
        +findAll() List~UserResponse~
        +findById(Long) UserResponse
        +create(UserRequest) UserResponse
        +update(Long, UserRequest) UserResponse
        +delete(Long) void
    }

    class UserServiceImpl {
        -UserRepository userRepository
        +findAll() List~UserResponse~
        +findById(Long) UserResponse
        +create(UserRequest) UserResponse
        +update(Long, UserRequest) UserResponse
        +delete(Long) void
        -findUserOrThrow(Long) User
        -toResponse(User) UserResponse
    }

    class ProjectService {
        <<interface>>
        +findAll() List~ProjectResponse~
        +findById(Long) ProjectResponse
        +create(ProjectRequest) ProjectResponse
        +update(Long, ProjectRequest) ProjectResponse
        +delete(Long) void
    }

    class ProjectServiceImpl {
        -ProjectRepository projectRepository
        +findAll() List~ProjectResponse~
        +findById(Long) ProjectResponse
        +create(ProjectRequest) ProjectResponse
        +update(Long, ProjectRequest) ProjectResponse
        +delete(Long) void
        -findProjectOrThrow(Long) Project
        -toResponse(Project) ProjectResponse
    }

    TaskServiceImpl ..|> TaskService : implements
    UserServiceImpl ..|> UserService : implements
    ProjectServiceImpl ..|> ProjectService : implements

    TaskServiceImpl --> TaskRepository : uses
    TaskServiceImpl --> UserRepository : uses
    TaskServiceImpl --> ProjectRepository : uses
    UserServiceImpl --> UserRepository : uses
    ProjectServiceImpl --> ProjectRepository : uses
```

---

## Backend — Controladores REST

Todos usan `@RestController` y delegan 100% en el servicio correspondiente.
Validacion de entrada via `@Valid` + Bean Validation.

```mermaid
classDiagram
    class TaskController {
        -TaskService taskService
        +getAll() ResponseEntity~List~TaskResponse~~
        +getById(Long) ResponseEntity~TaskResponse~
        +create(TaskRequest) ResponseEntity~TaskResponse~
        +update(Long, TaskRequest) ResponseEntity~TaskResponse~
        +updateStatus(Long, StatusUpdateRequest) ResponseEntity~TaskResponse~
        +delete(Long) ResponseEntity~Void~
    }

    class UserController {
        -UserService userService
        +getAll() ResponseEntity~List~UserResponse~~
        +getById(Long) ResponseEntity~UserResponse~
        +create(UserRequest) ResponseEntity~UserResponse~
        +update(Long, UserRequest) ResponseEntity~UserResponse~
        +delete(Long) ResponseEntity~Void~
    }

    class ProjectController {
        -ProjectService projectService
        +getAll() ResponseEntity~List~ProjectResponse~~
        +getById(Long) ResponseEntity~ProjectResponse~
        +create(ProjectRequest) ResponseEntity~ProjectResponse~
        +update(Long, ProjectRequest) ResponseEntity~ProjectResponse~
        +delete(Long) ResponseEntity~Void~
    }

    TaskController --> TaskService : delegates
    UserController --> UserService : delegates
    ProjectController --> ProjectService : delegates
```

---

## Backend — DTOs

Los DTOs de request llevan anotaciones Bean Validation. Los de response son POJOs planos.
No existe ningun DTO de Auth en el codigo actual (ver nota al final del documento).

```mermaid
classDiagram
    class TaskRequest {
        +String title
        +String description
        +TaskStatus status
        +Integer storyPoints
        +Double estimatedHours
        +LocalDate startDate
        +LocalDate endDate
        +Long assigneeId
        +Long projectId
    }

    class TaskResponse {
        +Long id
        +String title
        +String description
        +TaskStatus status
        +Integer storyPoints
        +Double estimatedHours
        +LocalDate startDate
        +LocalDate endDate
        +LocalDateTime createdAt
        +UserResponse assignee
        +ProjectResponse project
    }

    class UserRequest {
        +String username
        +String email
    }

    class UserResponse {
        +Long id
        +String username
        +String email
        +LocalDateTime createdAt
    }

    class ProjectRequest {
        +String name
        +String description
    }

    class ProjectResponse {
        +Long id
        +String name
        +String description
        +LocalDateTime createdAt
    }

    class StatusUpdateRequest {
        +TaskStatus status
    }

    class ErrorResponse {
        +LocalDateTime timestamp
        +int status
        +String error
        +String message
        +String path
    }

    TaskResponse --> UserResponse : embeds
    TaskResponse --> ProjectResponse : embeds
    TaskRequest --> TaskStatus : references
    StatusUpdateRequest --> TaskStatus : references
```

---

## Backend — Excepciones y manejo global

`GlobalExceptionHandler` captura las tres excepciones de dominio mas la generica
y devuelve siempre un `ErrorResponse` con el codigo HTTP apropiado.

```mermaid
classDiagram
    class RuntimeException {
        <<Java stdlib>>
    }

    class ResourceNotFoundException {
        +ResourceNotFoundException(String message)
        +ResourceNotFoundException(String resource, Long id)
    }

    class BusinessException {
        +BusinessException(String message)
    }

    class GlobalExceptionHandler {
        <<@RestControllerAdvice>>
        +handleResourceNotFound(ResourceNotFoundException, HttpServletRequest) ResponseEntity~ErrorResponse~
        +handleBusinessException(BusinessException, HttpServletRequest) ResponseEntity~ErrorResponse~
        +handleValidationErrors(MethodArgumentNotValidException, HttpServletRequest) ResponseEntity~ErrorResponse~
        +handleGenericException(Exception, HttpServletRequest) ResponseEntity~ErrorResponse~
    }

    ResourceNotFoundException --|> RuntimeException : extends
    BusinessException --|> RuntimeException : extends
    GlobalExceptionHandler ..> ErrorResponse : produces
    GlobalExceptionHandler ..> ResourceNotFoundException : handles
    GlobalExceptionHandler ..> BusinessException : handles
```

**Mapeo de excepciones a HTTP:**

| Excepcion | HTTP | `error` en body |
|---|---|---|
| `ResourceNotFoundException` | 404 | `Not Found` |
| `BusinessException` | 409 | `Conflict` |
| `MethodArgumentNotValidException` | 400 | `Bad Request` |
| `Exception` (cualquier otra) | 500 | `Internal Server Error` |

---

## Backend — Configuracion CORS

No hay `SecurityConfig` ni `JwtUtil` en el codigo actual. La unica configuracion de
infraestructura presente es `CorsConfig`.

```mermaid
classDiagram
    class CorsConfig {
        <<@Configuration>>
        -String allowedOriginsRaw
        +corsFilter() CorsFilter
    }
```

`allowedOriginsRaw` se inyecta desde la propiedad `cors.allowed-origins`
(default: `http://localhost:5173,http://localhost:3000`).
Metodos permitidos: GET, POST, PUT, PATCH, DELETE, OPTIONS. Credentials: true.

---

## Frontend — Tipos (`types.ts`)

Interfaces y tipos TypeScript que espeja el schema del backend.

```mermaid
classDiagram
    class TaskStatus {
        <<type alias>>
        TODO
        IN_PROGRESS
        DONE
    }

    class User {
        <<interface>>
        +number id
        +string username
        +string email
    }

    class Project {
        <<interface>>
        +number id
        +string name
        +string description
    }

    class Task {
        <<interface>>
        +number id
        +string title
        +string description
        +TaskStatus status
        +number|null storyPoints
        +number|null estimatedHours
        +string|null startDate
        +string|null endDate
        +User|null assignee
        +Project|null project
        +string createdAt
    }

    class TaskRequest {
        <<interface>>
        +string title
        +string description
        +TaskStatus status
        +number storyPoints
        +number estimatedHours
        +string startDate
        +string endDate
        +number assigneeId
        +number projectId
    }

    Task --> TaskStatus : status
    Task --> User : assignee (nullable)
    Task --> Project : project (nullable)
    TaskRequest --> TaskStatus : status
```

Nota: `User` y `Project` en el frontend no incluyen `createdAt`. El backend los
devuelve en `UserResponse` y `ProjectResponse`, pero `types.ts` no lo expone.

---

## Frontend — Modulo API (`api.ts`)

Cliente HTTP construido sobre `axios`. Base URL hardcodeada a `http://localhost:8080/api`.

```mermaid
classDiagram
    class ApiClient {
        <<axios instance>>
        +baseURL: http://localhost:8080/api
        +Content-Type: application/json
    }

    class TasksApi {
        <<exported functions>>
        +getTasks() Promise~Task[]~
        +createTask(TaskRequest) Promise~Task~
        +updateTaskStatus(number, TaskStatus) Promise~Task~
        +deleteTask(number) Promise~void~
    }

    class UsersApi {
        <<exported functions>>
        +getUsers() Promise~User[]~
        +createUser(username, email) Promise~User~
    }

    TasksApi --> ApiClient : GET /tasks
    TasksApi --> ApiClient : POST /tasks
    TasksApi --> ApiClient : PATCH /tasks/:id/status
    TasksApi --> ApiClient : DELETE /tasks/:id
    UsersApi --> ApiClient : GET /users
    UsersApi --> ApiClient : POST /users
```

El FE no implementa `getById`, `update` completo ni ningun endpoint de Projects
(aunque el backend los expone). Solo consume el subconjunto necesario para el tablero Kanban.

---

## Frontend — Componentes React

```mermaid
classDiagram
    class App {
        <<React.FC>>
        -Task[] tasks
        -boolean loading
        -string|null error
        -boolean showForm
        +fetchTasks() void
        +handleStatusChange(number, TaskStatus) void
        +handleDelete(number) void
        +handleCreated(Task) void
        +tasksByStatus(TaskStatus) Task[]
    }

    class BoardColumn {
        <<React.FC>>
        +string title
        +TaskStatus status
        +Task[] tasks
        +onStatusChange(number, TaskStatus) void
        +onDelete(number) void
    }

    class TaskCard {
        <<React.FC>>
        +Task task
        +onStatusChange(number, TaskStatus) void
        +onDelete(number) void
    }

    class CreateTaskForm {
        <<React.FC>>
        -string title
        -string description
        -string storyPoints
        -string estimatedHours
        -boolean submitting
        -string|null error
        +onCreated(Task) void
        +onClose() void
        +handleSubmit(FormEvent) void
    }

    App "1" --> "3" BoardColumn : renders (one per status)
    BoardColumn "1" --> "0..*" TaskCard : renders
    App --> CreateTaskForm : renders when showForm=true
    App ..> TasksApi : calls
    App ..> UsersApi : does not call (unused in current impl)
    CreateTaskForm ..> TasksApi : calls createTask
```

`LoginForm` y `RegisterForm` mencionados en la mision no existen en el codigo fuente del FE.

---

## Discrepancias entre la mision y el codigo real

Las siguientes clases/modulos estaban listados en la mision pero **no existen** en el codigo:

| Clase/modulo esperado | Ubicacion buscada | Veredicto |
|---|---|---|
| `AuthController` | `controller/` | No existe |
| `AuthServiceImpl` | `service/` | No existe |
| `JwtUtil` | cualquier paquete | No existe |
| `SecurityConfig` | `config/` | No existe. Solo existe `CorsConfig` |
| `AuthResponse` DTO | `dto/` | No existe |
| `LoginRequest` DTO | `dto/` | No existe |
| `RegisterRequest` DTO | `dto/` | No existe |
| `LoginForm` (FE) | `src/components/` | No existe |
| `RegisterForm` (FE) | `src/components/` | No existe |

El sistema **no implementa autenticacion** en el codigo actual. Los endpoints `/api/auth/*`
listados en la mision no tienen handler. Este documento refleja lo que esta en el codigo.
