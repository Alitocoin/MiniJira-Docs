# C4 Nivel 1 — Context

Mini Jira visto desde afuera: quién lo usa y con qué sistemas conversa.

## Diagrama

```mermaid
C4Context
  title Diagrama de contexto — Mini Jira

  Person(user, "Usuario", "PM o desarrollador que gestiona tareas y proyectos")

  System(minijira, "Mini Jira", "Sistema de gestión de tareas estilo Kanban. Permite crear proyectos, asignar tareas y mover tarjetas entre columnas TODO / IN_PROGRESS / DONE.")

  System_Ext(postgres, "PostgreSQL", "Base de datos relacional donde se persisten tareas, usuarios y proyectos.")

  Rel(user, minijira, "Gestiona tareas y proyectos", "HTTPS / navegador")
  Rel(minijira, postgres, "Lee y escribe datos", "JDBC / JPA")
```

## Descripción

Mini Jira es un sistema de gestión de proyectos liviano, inspirado en Jira. Permite a equipos de desarrollo organizar su trabajo en tableros Kanban con tres estados de tarea: **TODO**, **IN_PROGRESS** y **DONE**.

### Actores

| Actor | Descripción |
|-------|-------------|
| Usuario | PM, tech lead o desarrollador. Accede desde el navegador. No existe autenticación en la versión actual. |

### Sistemas externos

| Sistema | Rol |
|---------|-----|
| PostgreSQL | Única dependencia de infraestructura. Almacena las tres entidades del dominio: Task, User, Project. |

### Lo que Mini Jira NO hace (fuera de scope v1)

- Autenticación / autorización de usuarios.
- Notificaciones (email, Slack, etc.).
- Integraciones con repositorios Git.
- Reportes o dashboards analíticos.

---

Siguiente nivel: [02-container.md](02-container.md)
