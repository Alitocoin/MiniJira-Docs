# API REST — Resumen de endpoints

**Base URL:** `http://localhost:8080`
**Versión:** 1.0.0
**Tecnología:** Spring Boot 3.2 / Java 17
**Autenticación:** Ninguna (v1)
**Content-Type:** `application/json`

---

## Modelo de error estándar

Todos los errores devuelven el siguiente cuerpo:

```json
{
  "status": 404,
  "error": "Not Found",
  "message": "Tarea con id 99 no encontrada",
  "path": "/api/tasks/99",
  "timestamp": "2026-05-10T12:00:00",
  "details": []
}
```

El campo `details` solo está presente en errores 400 de validación y contiene la lista de campos fallidos.

---

## /api/tasks — Gestión de tareas

| # | Método | Path | Descripción | Request Body | Response |
|---|--------|------|-------------|--------------|----------|
| 1 | GET | `/api/tasks` | Lista todas las tareas con sus relaciones (usuario y proyecto asignados) | — | `200 OK` — Array de `TaskResponse` |
| 2 | GET | `/api/tasks/{id}` | Obtiene una tarea por ID | — | `200 OK` — `TaskResponse` / `404` si no existe |
| 3 | GET | `/api/tasks/status/{status}` | Filtra tareas por status. Valores válidos: `TODO`, `IN_PROGRESS`, `DONE` | — | `200 OK` — Array de `TaskResponse` |
| 4 | POST | `/api/tasks` | Crea una nueva tarea | `TaskRequest` | `201 Created` — `TaskResponse` / `400` si validación falla |
| 5 | PUT | `/api/tasks/{id}` | Reemplaza una tarea completa por ID | `TaskRequest` | `200 OK` — `TaskResponse` / `404` si no existe |
| 6 | PATCH | `/api/tasks/{id}/status` | Actualiza solo el status de una tarea | `TaskStatusRequest` | `200 OK` — `TaskResponse` / `404` si no existe |
| 7 | DELETE | `/api/tasks/{id}` | Elimina una tarea por ID | — | `204 No Content` / `404` si no existe |

### TaskRequest

```json
{
  "title": "Implementar login",
  "description": "Crear endpoint de autenticación JWT",
  "status": "TODO",
  "storyPoints": 3,
  "estimatedHours": 4.0,
  "startDate": "2026-05-10",
  "endDate": "2026-05-14",
  "assignedUserId": 1,
  "projectId": 1
}
```

Campos obligatorios: `title` (no vacío), `status` (no nulo). El resto es opcional.

### TaskStatusRequest

```json
{
  "status": "IN_PROGRESS"
}
```

Valores válidos: `TODO`, `IN_PROGRESS`, `DONE`.

### TaskResponse

```json
{
  "id": 1,
  "title": "Implementar login",
  "description": "Crear endpoint de autenticación JWT",
  "status": "TODO",
  "storyPoints": 3,
  "estimatedHours": 4.0,
  "startDate": "2026-05-10",
  "endDate": "2026-05-14",
  "assignedUser": {
    "id": 1,
    "name": "Ana García",
    "email": "ana@devforce.ai"
  },
  "project": {
    "id": 1,
    "name": "MiniJira",
    "description": "Proyecto de gestión de tareas"
  }
}
```

Los campos `assignedUser` y `project` pueden ser `null` si la tarea no tiene relación asignada.

---

## /api/projects — Gestión de proyectos

| # | Método | Path | Descripción | Request Body | Response |
|---|--------|------|-------------|--------------|----------|
| 1 | GET | `/api/projects` | Lista todos los proyectos | — | `200 OK` — Array de `ProjectResponse` |
| 2 | GET | `/api/projects/{id}` | Obtiene un proyecto por ID | — | `200 OK` — `ProjectResponse` / `404` si no existe |
| 3 | POST | `/api/projects` | Crea un nuevo proyecto | `ProjectRequest` | `201 Created` — `ProjectResponse` / `400` si validación falla |

### ProjectRequest

```json
{
  "name": "MiniJira",
  "description": "Sistema de gestión de tareas estilo Kanban"
}
```

Campo obligatorio: `name` (no vacío). `description` es opcional.

### ProjectResponse

```json
{
  "id": 1,
  "name": "MiniJira",
  "description": "Sistema de gestión de tareas estilo Kanban"
}
```

---

## /api/users — Gestión de usuarios

| # | Método | Path | Descripción | Request Body | Response |
|---|--------|------|-------------|--------------|----------|
| 1 | GET | `/api/users` | Lista todos los usuarios | — | `200 OK` — Array de `UserResponse` |
| 2 | GET | `/api/users/{id}` | Obtiene un usuario por ID | — | `200 OK` — `UserResponse` / `404` si no existe |
| 3 | POST | `/api/users` | Crea un nuevo usuario. Falla si el email ya existe. | `UserRequest` | `201 Created` — `UserResponse` / `400` si validación falla / `409` si email duplicado |

### UserRequest

```json
{
  "name": "Ana García",
  "email": "ana@devforce.ai"
}
```

Campos obligatorios: `name` (no vacío), `email` (no vacío, formato email válido). El email debe ser único en el sistema.

### UserResponse

```json
{
  "id": 1,
  "name": "Ana García",
  "email": "ana@devforce.ai"
}
```

### Error 409 — Email duplicado

```json
{
  "status": 409,
  "error": "Conflict",
  "message": "Ya existe un usuario con el email: ana@devforce.ai",
  "path": "/api/users",
  "timestamp": "2026-05-10T12:00:00"
}
```

---

## CORS

La API acepta requests desde cualquier origen en todas las rutas `/api/**`. Métodos permitidos: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`. El origen se puede restringir via la variable de entorno `cors.allowed-origins`.
