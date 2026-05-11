# Data Dictionary — MiniJira

Diccionario de datos inferido de las entidades JPA del repositorio `MiniJira-BE` (branch `development`).
Motor de base de datos: **PostgreSQL** (gestionado por Hibernate / Spring Data JPA).

Las columnas, tipos y constraints se derivan directamente de las anotaciones JPA presentes
en `User.java`, `Task.java` y `Project.java`. Los tipos SQL son los que Hibernate genera
para PostgreSQL con estas anotaciones.

---

## Tabla: `users`

**Proposito**: almacena las cuentas de usuario del sistema. Se usa como referencia en
la relacion `tasks.assignee_id`. No hay campo de password ni de rol: el sistema actual
no implementa autenticacion.

**Fuente JPA**: `com.minijira.backend.model.User`

### Columnas

| Columna | Tipo SQL | Nullable | Unique | Default | Descripcion |
|---|---|---|---|---|---|
| `id` | `bigint` | NO | SI (PK) | autoincrement | Clave primaria generada por secuencia de la BD |
| `username` | `varchar(100)` | NO | SI | — | Nombre de usuario. Unicidad validada tambien en `UserServiceImpl` |
| `email` | `varchar(255)` | NO | SI | — | Correo electronico. Unicidad validada tambien en `UserServiceImpl` |
| `created_at` | `timestamp` | NO | NO | `now()` via `@PrePersist` | Momento de creacion del registro. No actualizable (`updatable = false`) |

### Relaciones

| Tipo | Tabla relacionada | Columna FK | Descripcion |
|---|---|---|---|
| Referenciada por | `tasks` | `assignee_id` | Un usuario puede estar asignado a muchas tareas |

### Indices

| Nombre | Columnas | Tipo |
|---|---|---|
| `users_pkey` | `id` | PRIMARY KEY |
| `users_username_key` | `username` | UNIQUE (generado por `@Column(unique=true)`) |
| `users_email_key` | `email` | UNIQUE (generado por `@Column(unique=true)`) |

---

## Tabla: `projects`

**Proposito**: agrupa tareas bajo un proyecto. Una tarea puede pertenecer a un proyecto
(o a ninguno). El proyecto no tiene estado propio ni fechas.

**Fuente JPA**: `com.minijira.backend.model.Project`

### Columnas

| Columna | Tipo SQL | Nullable | Unique | Default | Descripcion |
|---|---|---|---|---|---|
| `id` | `bigint` | NO | SI (PK) | autoincrement | Clave primaria generada por secuencia de la BD |
| `name` | `varchar(200)` | NO | NO | — | Nombre del proyecto |
| `description` | `text` | SI | NO | `NULL` | Descripcion larga sin limite de caracteres |
| `created_at` | `timestamp` | NO | NO | `now()` via `@PrePersist` | Momento de creacion del registro. No actualizable (`updatable = false`) |

### Relaciones

| Tipo | Tabla relacionada | Columna FK | Descripcion |
|---|---|---|---|
| Referenciada por | `tasks` | `project_id` | Un proyecto puede tener muchas tareas |

### Indices

| Nombre | Columnas | Tipo |
|---|---|---|
| `projects_pkey` | `id` | PRIMARY KEY |

---

## Tabla: `tasks`

**Proposito**: entidad central del sistema. Representa una tarea del tablero Kanban con
estado, estimaciones, fechas y referencias opcionales a usuario asignado y proyecto.

**Fuente JPA**: `com.minijira.backend.model.Task`

### Columnas

| Columna | Tipo SQL | Nullable | Unique | Default | Descripcion |
|---|---|---|---|---|---|
| `id` | `bigint` | NO | SI (PK) | autoincrement | Clave primaria generada por secuencia de la BD |
| `title` | `varchar(300)` | NO | NO | — | Titulo de la tarea. Maximo 300 caracteres (validado en Bean Validation tambien) |
| `description` | `text` | SI | NO | `NULL` | Descripcion detallada sin limite de caracteres |
| `status` | `varchar(20)` | NO | NO | `'TODO'` via `@PrePersist` | Estado del Kanban. Valores posibles: `TODO`, `IN_PROGRESS`, `DONE` (almacenado como string por `@Enumerated(STRING)`) |
| `story_points` | `integer` | SI | NO | `NULL` | Estimacion en puntos de historia |
| `estimated_hours` | `float8` (double) | SI | NO | `NULL` | Estimacion en horas. Acepta decimales (ej. 4.5) |
| `start_date` | `date` | SI | NO | `NULL` | Fecha de inicio planificada |
| `end_date` | `date` | SI | NO | `NULL` | Fecha de vencimiento planificada |
| `created_at` | `timestamp` | NO | NO | `now()` via `@PrePersist` | Momento de creacion. No actualizable (`updatable = false`) |
| `assignee_id` | `bigint` | SI | NO | `NULL` | FK a `users.id`. Null = sin asignar. Carga LAZY |
| `project_id` | `bigint` | SI | NO | `NULL` | FK a `projects.id`. Null = sin proyecto. Carga LAZY |

### Relaciones

| Tipo | Tabla relacionada | Columna FK local | On Delete | Descripcion |
|---|---|---|---|---|
| ManyToOne | `users` | `assignee_id` | Sin configurar (Hibernate default: restrict) | Usuario asignado a la tarea (opcional) |
| ManyToOne | `projects` | `project_id` | Sin configurar (Hibernate default: restrict) | Proyecto al que pertenece la tarea (opcional) |

> Nota: el codigo JPA no define `cascade` ni `orphanRemoval` en estas relaciones. El comportamiento
> de borrado en cascada no esta configurado explicitamente. Borrar un `User` o `Project` que
> tenga tareas asociadas fallara con constraint de FK a nivel de base de datos.

### Indices

| Nombre | Columnas | Tipo |
|---|---|---|
| `tasks_pkey` | `id` | PRIMARY KEY |
| *(generado por JPA)* | `assignee_id` | INDEX de FK |
| *(generado por JPA)* | `project_id` | INDEX de FK |

### Consultas derivadas disponibles en `TaskRepository`

Ademas de las operaciones CRUD estandar, el repositorio expone:

| Metodo | SQL equivalente |
|---|---|
| `findByStatus(TaskStatus)` | `SELECT * FROM tasks WHERE status = ?` |
| `findByProjectId(Long)` | `SELECT * FROM tasks WHERE project_id = ?` |
| `findByAssigneeId(Long)` | `SELECT * FROM tasks WHERE assignee_id = ?` |

Estos metodos no son invocados actualmente desde los servicios ni controladores;
estan disponibles pero sin usar en el codigo de la branch `development`.

---

## Diagrama ER simplificado

```
users
  id PK
  username UNIQUE NOT NULL
  email UNIQUE NOT NULL
  created_at NOT NULL

projects
  id PK
  name NOT NULL
  description NULL
  created_at NOT NULL

tasks
  id PK
  title NOT NULL
  description NULL
  status NOT NULL
  story_points NULL
  estimated_hours NULL
  start_date NULL
  end_date NULL
  created_at NOT NULL
  assignee_id NULL --> users.id
  project_id NULL --> projects.id
```
