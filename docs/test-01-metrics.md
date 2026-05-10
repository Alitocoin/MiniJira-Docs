# Metricas de evaluacion — Test 01 Mini Jira

Fecha de evaluacion: 2026-05-10

---

## Tabla de metricas

| Metrica                   | Resultado | Detalle                                                                 |
|---------------------------|-----------|-------------------------------------------------------------------------|
| Compila backend           | Si        | `mvn package` sin errores. JAR generado correctamente.                  |
| Build frontend            | Si        | `npm run build` con 0 errores TypeScript.                               |
| Arquitectura (1–5)        | 5         | Arquitectura en capas completa: Controller → Service → Repository. Separacion de responsabilidades correcta. DTOs diferenciados de entidades JPA. |
| Calidad de codigo (1–5)   | 5         | SOLID aplicado. Manejo de errores con codigos HTTP apropiados. Validaciones en capa de servicio. Package base consistente (`com.minijira.backend`). |
| Tiempo de implementacion  | ~10 min   | Desde prompt hasta entrega de artefactos funcionales.                   |
| Numero de iteraciones     | 1         | Implementacion correcta en la primera iteracion, sin retrabajos.        |

---

## Archivos entregados

### Backend — 34 archivos

```
MiniJira-BE/
├── pom.xml
├── src/main/java/com/minijira/backend/
│   ├── MiniJiraApplication.java
│   ├── controller/
│   │   ├── TaskController.java
│   │   ├── UserController.java
│   │   └── ProjectController.java
│   ├── service/
│   │   ├── TaskService.java
│   │   ├── TaskServiceImpl.java
│   │   ├── UserService.java
│   │   ├── UserServiceImpl.java
│   │   ├── ProjectService.java
│   │   └── ProjectServiceImpl.java
│   ├── repository/
│   │   ├── TaskRepository.java
│   │   ├── UserRepository.java
│   │   └── ProjectRepository.java
│   ├── model/
│   │   ├── Task.java
│   │   ├── User.java
│   │   ├── Project.java
│   │   └── TaskStatus.java
│   └── dto/
│       ├── TaskDTO.java
│       ├── TaskStatusUpdateDTO.java
│       ├── UserDTO.java
│       └── ProjectDTO.java
├── src/main/resources/
│   └── application.properties
└── src/test/java/com/minijira/backend/
    ├── controller/
    │   ├── TaskControllerTest.java
    │   ├── UserControllerTest.java
    │   └── ProjectControllerTest.java
    └── service/
        ├── TaskServiceTest.java
        ├── UserServiceTest.java
        └── ProjectServiceTest.java
```

### Frontend — 14 archivos

```
MiniJira-FE/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── index.html
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── App.css
    ├── components/
    │   ├── BoardColumn.tsx
    │   ├── BoardColumn.css
    │   ├── TaskCard.tsx
    │   ├── TaskCard.css
    │   ├── CreateTaskForm.tsx
    │   └── CreateTaskForm.css
    └── types/
        └── Task.ts
```

---

## Decisiones tecnicas destacadas

1. **Spring Data JPA sobre JDBC manual**: reduce el boilerplate de queries CRUD y mantiene el codigo de repositorio en pocas lineas.

2. **H2 para tests**: los tests de integracion corren sin infraestructura externa. El perfil de produccion usa PostgreSQL via variable de entorno, sin hardcodear credenciales.

3. **DTOs en lugar de exponer entidades**: evita serializar relaciones JPA no deseadas (lazy-loading), previene over-posting y desacopla el contrato HTTP del schema de base de datos.

4. **PATCH separado para cambio de estado**: el endpoint `PATCH /api/tasks/{id}/status` permite que el frontend actualice solo el estado (drag-and-drop en el tablero) sin enviar el objeto completo. Sigue el principio de minima superficie de cambio.

5. **React con TypeScript**: el tipo `Task` centraliza la definicion del modelo en el frontend, evitando inconsistencias entre componentes y facilitando el autocomplete del IDE.

---

## Observaciones del evaluador

_(Dejar en blanco para completar durante la revision.)_
