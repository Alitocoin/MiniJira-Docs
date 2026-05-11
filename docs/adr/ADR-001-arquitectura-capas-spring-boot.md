# ADR-001 — Arquitectura en capas para el backend Spring Boot

**Estado:** Aceptado
**Fecha:** 2026-05-11
**Autores:** Carlos Frost (Backend), Luis Barrios (Docs)
**Relacionado con:** implementación de Test 01 (fullstack base)

---

## Contexto

Al comenzar la implementación del backend de MiniJira debíamos decidir cómo organizar el código de servidor. Las opciones consideradas fueron:

1. **Arquitectura plana** — controllers con lógica de negocio embebida directamente.
2. **Arquitectura en capas** — Controller → Service (interface + implementación) → Repository, con DTOs separados de las entidades JPA.
3. **Arquitectura hexagonal (ports & adapters)** — mayor aislamiento pero complejidad alta para el tamaño del proyecto.

La restricción principal era que el proyecto tiene alcance acotado (herramienta de gestión de tareas tipo Jira) y el equipo de backend era de una sola persona.

---

## Decisión

Se adoptó la **arquitectura en 3 capas** con las siguientes convenciones:

- **Controller** — recibe la petición HTTP, delega en el service, devuelve la respuesta HTTP. Sin lógica de negocio.
- **Service** — interfaz pública (`TaskService`, `UserService`, `ProjectService`) más una implementación (`*ServiceImpl`). Contiene toda la lógica de negocio.
- **Repository** — extiende `JpaRepository<T, ID>`. Sin SQL manual salvo cuando sea estrictamente necesario.
- **DTOs separados de entidades** — `TaskRequest` / `TaskResponse`, `UserRequest` / `UserResponse`, etc. Las entidades JPA (`Task`, `User`, `Project`) nunca se exponen directamente en la API.

### Estructura de paquetes resultante

```
com.minijira.backend
├── controller/     TaskController, UserController, ProjectController, AuthController
├── service/        TaskService (interface), TaskServiceImpl, UserService, ...
├── repository/     TaskRepository, UserRepository, ProjectRepository
├── model/          Task, User, Project, TaskStatus (enum)
├── dto/            TaskRequest, TaskResponse, UserRequest, UserResponse, ...
├── exception/      ResourceNotFoundException, BusinessException
└── config/         CorsConfig, SecurityConfig
```

---

## Consecuencias

**Positivas:**
- La interfaz de service permite mockear dependencias en tests unitarios con Mockito sin necesidad de levantar el contexto de Spring (`@ExtendWith(MockitoExtension.class)`). Esto fue decisivo para escribir los 27 tests de `TaskServiceImplTest` y `UserServiceImplTest`.
- La separación Controller / Service / Repository facilita cambiar la capa de persistencia sin tocar la lógica de negocio.
- Los DTOs protegen el modelo interno de la API: agregar campos a la entidad no rompe el contrato expuesto.

**Negativas:**
- Mayor cantidad de clases por funcionalidad (interface + impl + DTO request + DTO response). Para un proyecto pequeño puede sentirse verboso.
- El mapeo manual entidad → DTO requiere código repetitivo. No se incorporó MapStruct para mantener las dependencias simples.

**Neutral:**
- `ProjectServiceImpl` no tiene tests unitarios todavía (gap documentado en QA Test Plan). La arquitectura lo facilita cuando se implemente.
