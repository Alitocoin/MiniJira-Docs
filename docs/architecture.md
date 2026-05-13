# Arquitectura — Mini Jira

Documento de arquitectura usando el modelo C4 (Simon Brown). Niveles L1 y L2 obligatorios; L3 para el container API por su complejidad interna.

---

## L1 — Diagrama de contexto

Muestra quiénes usan Mini Jira y con qué sistemas externos interactúa.

```mermaid
C4Context
  title Diagrama de contexto — Mini Jira

  Person(dev, "Desarrollador / PM", "Gestiona tareas y proyectos del equipo")

  System_Boundary(mj, "Mini Jira") {
    System(minijira, "Mini Jira", "Sistema Kanban de gestion de tareas")
  }

  System_Ext(postgres, "PostgreSQL", "Base de datos relacional (produccion)")
  System_Ext(h2, "H2 In-Memory", "Base de datos para tests automatizados")

  Rel(dev, minijira, "Crea tareas, cambia estados, asigna usuarios", "HTTPS / Browser")
  Rel(minijira, postgres, "Persiste datos en produccion", "JDBC")
  Rel(minijira, h2, "Usa en ejecucion de tests", "JDBC en memoria")
```

**Lectura**: un desarrollador o PM accede al sistema a través del navegador. Mini Jira persiste toda la información en PostgreSQL (producción) y usa H2 para los tests automáticos sin infraestructura externa.

---

## L2 — Diagrama de containers

Zoom dentro del sistema Mini Jira. Cada caja es un proceso desplegable independientemente.

```mermaid
C4Container
  title Diagrama de containers — Mini Jira

  Person(dev, "Desarrollador / PM", "Usuario del sistema")

  System_Boundary(mj, "Mini Jira") {
    Container(fe, "SPA Frontend", "React 18 + TypeScript + Vite", "Tablero Kanban. Columnas TODO / IN_PROGRESS / DONE. Formulario de creacion de tareas.")
    Container(api, "API Backend", "Spring Boot 3.x / Java 17", "REST API. Expone recursos Task, User, Project. Valida, procesa y persiste.")
    ContainerDb(db, "Base de datos", "PostgreSQL 14+", "Tablas: tasks, users, projects. FK entre tasks y users/projects.")
  }

  Rel(dev, fe, "Accede via navegador", "HTTPS :5173")
  Rel(fe, api, "Llamadas REST/JSON", "HTTP :8080/api")
  Rel(api, db, "Consultas y escrituras", "JDBC / Spring Data JPA")
```

**Decisiones clave de este nivel**:

- **SPA desacoplada del backend**: el frontend es un artefacto Vite independiente. En producción se puede servir desde un CDN o Nginx sin tocar el JAR de Spring Boot.
- **API stateless**: el backend no mantiene sesiones. Cada request lleva toda la información necesaria. Facilita el escalado horizontal.
- **PostgreSQL en producción, H2 en tests**: sin necesidad de levantar una base real para correr `mvn test`. Ver `adr/` si se agrega en el futuro.

---

## L3 — Diagrama de componentes: API Backend

Zoom dentro del container "API Backend". Muestra la arquitectura en capas.

```mermaid
C4Component
  title Diagrama de componentes — API Backend (Spring Boot)

  Container_Boundary(api, "API Backend") {
    Component(taskCtrl, "TaskController", "Spring @RestController", "Endpoints REST para /api/tasks. Recibe HTTP, delega en servicio.")
    Component(userCtrl, "UserController", "Spring @RestController", "Endpoints REST para /api/users.")
    Component(projCtrl, "ProjectController", "Spring @RestController", "Endpoints REST para /api/projects.")

    Component(taskSvc, "TaskService", "Spring @Service", "Logica de negocio de tareas. Valida transiciones de estado. Resuelve relaciones.")
    Component(userSvc, "UserService", "Spring @Service", "Logica de negocio de usuarios.")
    Component(projSvc, "ProjectService", "Spring @Service", "Logica de negocio de proyectos.")

    Component(taskRepo, "TaskRepository", "Spring Data JPA @Repository", "CRUD sobre la tabla tasks.")
    Component(userRepo, "UserRepository", "Spring Data JPA @Repository", "CRUD sobre la tabla users.")
    Component(projRepo, "ProjectRepository", "Spring Data JPA @Repository", "CRUD sobre la tabla projects.")
  }

  ContainerDb(db, "PostgreSQL", "PostgreSQL 14+", "")

  Rel(taskCtrl, taskSvc, "llama")
  Rel(userCtrl, userSvc, "llama")
  Rel(projCtrl, projSvc, "llama")

  Rel(taskSvc, taskRepo, "usa")
  Rel(taskSvc, userRepo, "resuelve assignee")
  Rel(taskSvc, projRepo, "resuelve project")
  Rel(userSvc, userRepo, "usa")
  Rel(projSvc, projRepo, "usa")

  Rel(taskRepo, db, "JDBC")
  Rel(userRepo, db, "JDBC")
  Rel(projRepo, db, "JDBC")
```

**Responsabilidades por capa**:

| Capa        | Responsabilidad                                                    |
|-------------|--------------------------------------------------------------------|
| Controller  | Recibir y parsear HTTP. Delegar. No contiene logica de negocio.    |
| Service     | Validaciones, reglas de negocio, orquestacion entre repositorios.  |
| Repository  | Acceso a datos via Spring Data JPA. Sin logica de negocio.         |
| Model/DTO   | Entidades JPA para persistencia; DTOs para el contrato HTTP.       |

---

## Modelo de datos

```
+------------------+       +------------------+       +------------------+
|     projects     |       |      tasks       |       |      users       |
+------------------+       +------------------+       +------------------+
| id (PK)          |<--+   | id (PK)          |   +-->| id (PK)          |
| name             |   |   | title            |   |   | name             |
| description      |   |   | description      |   |   | email (unique)   |
+------------------+   |   | status           |   |   +------------------+
                        |   | story_points     |   |
                        |   | estimated_hours  |   |
                        |   | start_date       |   |
                        |   | end_date         |   |
                        +---| project_id (FK)  |   |
                            | assignee_id (FK) |---+
                            +------------------+
```

**Enum Task.status**: `TODO` | `IN_PROGRESS` | `DONE`

Relaciones:
- Una tarea pertenece a exactamente un proyecto (`project_id NOT NULL`).
- Una tarea puede tener un asignado opcional (`assignee_id NULL`).
- Un usuario puede tener muchas tareas asignadas.
- Un proyecto puede tener muchas tareas.

---

## Principios aplicados

### SOLID

| Principio | Aplicacion concreta                                                                 |
|-----------|-------------------------------------------------------------------------------------|
| SRP       | Cada clase tiene una sola razon de cambio: controller solo maneja HTTP, service solo logica. |
| OCP       | Los repositorios extienden `JpaRepository` sin modificar su contrato.               |
| LSP       | Las interfaces de servicio pueden intercambiarse por implementaciones mock en tests.|
| ISP       | Interfaces de repositorio granulares por entidad, no un repositorio "dios".        |
| DIP       | Controllers dependen de interfaces de servicio, no de implementaciones concretas.   |

### Separacion de concerns (DTOs vs Entidades)

Las entidades JPA (`@Entity`) no se exponen directamente en la API. Los DTOs definen el contrato del endpoint independientemente de la estructura de la base de datos. Esto protege contra over-posting y permite evolucionar el schema sin romper clientes.

### Tests con H2

El perfil de test usa `spring.datasource.url=jdbc:h2:mem:testdb`. El schema se regenera en cada ejecucion (`ddl-auto=create-drop`). Ningun test necesita infraestructura externa.
