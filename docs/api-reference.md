# API Reference — Mini Jira

Base URL: `http://localhost:8080/api`

Todos los endpoints aceptan y devuelven `Content-Type: application/json`.

---

## Tareas

### GET /api/tasks

Lista todas las tareas registradas en el sistema.

**Response 200**

```json
[
  {
    "id": 1,
    "title": "Diseñar modelo de datos",
    "description": "Definir entidades y relaciones principales",
    "status": "DONE",
    "storyPoints": 3,
    "estimatedHours": 4,
    "startDate": "2026-05-01",
    "endDate": "2026-05-02",
    "projectId": 1,
    "assigneeId": 2
  }
]
```

---

### GET /api/tasks/{id}

Obtiene una tarea por su ID.

**Path params**

| Param | Tipo   | Descripcion     |
|-------|--------|-----------------|
| id    | Long   | ID de la tarea  |

**Response 200**

```json
{
  "id": 1,
  "title": "Diseñar modelo de datos",
  "description": "Definir entidades y relaciones principales",
  "status": "DONE",
  "storyPoints": 3,
  "estimatedHours": 4,
  "startDate": "2026-05-01",
  "endDate": "2026-05-02",
  "projectId": 1,
  "assigneeId": 2
}
```

**Errores**

| Codigo | Descripcion                   |
|--------|-------------------------------|
| 404    | Tarea no encontrada           |

---

### POST /api/tasks

Crea una nueva tarea.

**Request body**

```json
{
  "title": "Implementar endpoint de login",
  "description": "POST /api/auth/login con JWT",
  "status": "TODO",
  "storyPoints": 5,
  "estimatedHours": 6,
  "startDate": "2026-05-10",
  "endDate": "2026-05-12",
  "projectId": 1,
  "assigneeId": 3
}
```

| Campo          | Tipo    | Requerido | Descripcion                              |
|----------------|---------|-----------|------------------------------------------|
| title          | String  | Si        | Titulo de la tarea                       |
| description    | String  | No        | Descripcion detallada                    |
| status         | Enum    | Si        | `TODO`, `IN_PROGRESS` o `DONE`           |
| storyPoints    | Integer | No        | Puntos de historia (escala Fibonacci)    |
| estimatedHours | Double  | No        | Horas estimadas de trabajo               |
| startDate      | Date    | No        | Fecha de inicio (`YYYY-MM-DD`)           |
| endDate        | Date    | No        | Fecha de fin (`YYYY-MM-DD`)              |
| projectId      | Long    | Si        | ID del proyecto al que pertenece         |
| assigneeId     | Long    | No        | ID del usuario asignado                  |

**Response 201**

```json
{
  "id": 7,
  "title": "Implementar endpoint de login",
  "description": "POST /api/auth/login con JWT",
  "status": "TODO",
  "storyPoints": 5,
  "estimatedHours": 6,
  "startDate": "2026-05-10",
  "endDate": "2026-05-12",
  "projectId": 1,
  "assigneeId": 3
}
```

**Errores**

| Codigo | Descripcion                            |
|--------|----------------------------------------|
| 400    | Body inválido o campos requeridos ausentes |
| 404    | projectId o assigneeId no existe       |

---

### PUT /api/tasks/{id}

Actualiza todos los campos de una tarea existente.

**Path params**

| Param | Tipo | Descripcion    |
|-------|------|----------------|
| id    | Long | ID de la tarea |

**Request body** — misma estructura que POST /api/tasks.

**Response 200** — tarea actualizada completa.

**Errores**

| Codigo | Descripcion                   |
|--------|-------------------------------|
| 400    | Body invalido                 |
| 404    | Tarea no encontrada           |

---

### PATCH /api/tasks/{id}/status

Cambia solo el estado de una tarea. Util para drag-and-drop en el tablero.

**Path params**

| Param | Tipo | Descripcion    |
|-------|------|----------------|
| id    | Long | ID de la tarea |

**Request body**

```json
{
  "status": "IN_PROGRESS"
}
```

| Campo  | Tipo | Valores aceptados              |
|--------|------|--------------------------------|
| status | Enum | `TODO`, `IN_PROGRESS`, `DONE`  |

**Response 200**

```json
{
  "id": 7,
  "status": "IN_PROGRESS"
}
```

**Errores**

| Codigo | Descripcion                  |
|--------|------------------------------|
| 400    | Estado invalido              |
| 404    | Tarea no encontrada          |

---

### DELETE /api/tasks/{id}

Elimina una tarea por su ID.

**Path params**

| Param | Tipo | Descripcion    |
|-------|------|----------------|
| id    | Long | ID de la tarea |

**Response 204** — sin cuerpo.

**Errores**

| Codigo | Descripcion         |
|--------|---------------------|
| 404    | Tarea no encontrada |

---

## Usuarios

### GET /api/users

Lista todos los usuarios.

**Response 200**

```json
[
  {
    "id": 1,
    "name": "Ana Torres",
    "email": "ana@minijira.dev"
  }
]
```

---

### POST /api/users

Crea un usuario nuevo.

**Request body**

```json
{
  "name": "Carlos Mendez",
  "email": "carlos@minijira.dev"
}
```

| Campo | Tipo   | Requerido | Descripcion             |
|-------|--------|-----------|-------------------------|
| name  | String | Si        | Nombre completo         |
| email | String | Si        | Email unico del usuario |

**Response 201**

```json
{
  "id": 4,
  "name": "Carlos Mendez",
  "email": "carlos@minijira.dev"
}
```

**Errores**

| Codigo | Descripcion                   |
|--------|-------------------------------|
| 400    | Email invalido o duplicado    |

---

## Proyectos

### GET /api/projects

Lista todos los proyectos.

**Response 200**

```json
[
  {
    "id": 1,
    "name": "Mini Jira v1",
    "description": "MVP del sistema de gestion de tareas"
  }
]
```

---

### POST /api/projects

Crea un proyecto nuevo.

**Request body**

```json
{
  "name": "Mini Jira v2",
  "description": "Segunda iteracion con autenticacion JWT"
}
```

| Campo       | Tipo   | Requerido | Descripcion            |
|-------------|--------|-----------|------------------------|
| name        | String | Si        | Nombre del proyecto    |
| description | String | No        | Descripcion del alcance |

**Response 201**

```json
{
  "id": 2,
  "name": "Mini Jira v2",
  "description": "Segunda iteracion con autenticacion JWT"
}
```

**Errores**

| Codigo | Descripcion                    |
|--------|--------------------------------|
| 400    | Nombre ausente o duplicado     |
