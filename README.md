# MiniJira-Docs

Repositorio de documentación del proyecto **Mini Jira** — sistema de gestión de tareas y proyectos desarrollado por el equipo DevForce.

## Estructura

```
MiniJira-DevForce01/
└── Docs/
    ├── Architecture/
    │   └── C4/
    │       ├── 01-context.md        Diagrama C4 Nivel 1: visión del sistema desde afuera
    │       ├── 02-container.md      Diagrama C4 Nivel 2: contenedores desplegables
    │       └── 03-component.md      Diagrama C4 Nivel 3: componentes internos del backend
    ├── QA/
    │   └── 2026/05/10/
    │       └── qa-report.md         Reporte de calidad: tests unitarios e integración
    ├── Evaluation/
    │   └── 2026/05/10/
    │       └── metrics-report.md    Informe de métricas del Test 01 de evaluación IA
    └── API/
        └── openapi-summary.md       Resumen de los 13 endpoints REST del sistema
```

## Repositorios del proyecto

| Repo | Descripción | Branch principal |
|------|-------------|-----------------|
| MiniJira-BE | Spring Boot 3.2 / Java 17 — API REST | feat/mini-jira-backend |
| MiniJira-FE | React 18 + Vite — SPA Kanban | feat/mini-jira-frontend |
| MiniJira-Docs | Esta documentación | feat/mini-jira-docs |

## Resumen del sistema

Mini Jira es una herramienta Kanban que permite gestionar tareas organizadas en columnas (TODO / IN_PROGRESS / DONE), asignarlas a usuarios y asociarlas a proyectos.

El sistema consta de tres piezas:
- **React SPA** — interfaz Kanban, corre en el puerto 5173.
- **Spring Boot API** — 13 endpoints REST, corre en el puerto 8080.
- **PostgreSQL** — base de datos relacional, corre en el puerto 5432.

Para ver cómo está estructurado internamente cada componente, empezá por [Architecture/C4/01-context.md](MiniJira-DevForce01/Docs/Architecture/C4/01-context.md).

## Equipo

| Rol | Persona |
|-----|---------|
| PM / Tech Lead | Jose Perez |
| Backend | Carlos Frost |
| Frontend | Paula Aguilar |
| QA | Juan Mendez |
| Docs | John Lee |
