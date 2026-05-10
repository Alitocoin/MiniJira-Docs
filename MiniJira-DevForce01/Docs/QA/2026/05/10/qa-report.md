# Reporte QA — Mini Jira

**Fecha:** 2026-05-10
**Responsable QA:** Juan Mendez
**Versión del sistema:** 1.0.0
**Rama evaluada:** feat/mini-jira-backend

---

## 1. Resumen ejecutivo

El backend de Mini Jira cuenta con **29 tests automatizados**: 15 unitarios y 14 de integración, más 1 test de carga de contexto. El foco está en la capa `Task`, que concentra la mayor complejidad de negocio del sistema (relaciones con User y Project, filtrado por status, actualización parcial). La cobertura global estimada es del **40–45%**, con la capa Task cubierta al **85–90%**. Los servicios `UserService` y `ProjectService` tienen cobertura baja y constituyen el principal gap de calidad.

Los tests fueron validados estructuralmente. El entorno sandbox no tiene `mvn` ni `java` instalados, por lo que no se ejecutaron contra el binario compilado.

---

## 2. Estrategia de testing

| Capa | Herramienta | Enfoque |
|------|-------------|---------|
| Service (unit) | JUnit 5 + Mockito (`MockitoExtension`) | Mocks de repositories, sin base de datos |
| Controller (integración) | JUnit 5 + `@WebMvcTest` + MockMvc | Slice test del MVC; `TaskService` mockeado |
| Application (smoke) | `@SpringBootTest` + H2 en memoria | Verifica que el contexto Spring levanta correctamente |

**Principios aplicados:**
- Tests con nombre descriptivo via `@DisplayName`.
- Setup centralizado en `@BeforeEach` para evitar duplicación.
- Verificación de interacciones con `verify()` además de asserts de resultado.
- Casos positivos y negativos cubiertos para cada operación.

---

## 3. Tests implementados

### 3.1 TaskServiceTest (15 tests unitarios — MockitoExtension)

| # | Nombre del test | Tipo | Resultado esperado |
|---|-----------------|------|--------------------|
| 1 | `createTask_shouldReturnTaskResponse_whenValidRequest` | Unitario | TaskResponse con todos los campos, incluidas relaciones User y Project |
| 2 | `createTask_sinRelaciones_shouldReturnTaskResponseBasica` | Unitario | TaskResponse con assignedUser y project null |
| 3 | `createTask_conUsuarioInexistente_shouldThrowResourceNotFoundException` | Unitario | `ResourceNotFoundException` con ID 99 |
| 4 | `getTaskById_shouldReturnTaskResponse_whenExists` | Unitario | TaskResponse con id=1 y title correcto |
| 5 | `getTaskById_shouldThrowException_whenNotFound` | Unitario | `ResourceNotFoundException` con ID 999 |
| 6 | `updateTaskStatus_shouldChangeStatus_whenValidStatus` | Unitario | Status cambiado a IN_PROGRESS, `save()` llamado una vez |
| 7 | `updateTaskStatus_shouldThrowException_whenTaskNotFound` | Unitario | `ResourceNotFoundException`, `save()` nunca llamado |
| 8 | `updateTaskStatus_shouldTransitionFromTodoDone` | Unitario | Status cambiado a DONE |
| 9 | `getAllTasks_shouldReturnList` | Unitario | Lista de 2 tareas, `findAllWithRelations()` llamado una vez |
| 10 | `getAllTasks_shouldReturnEmptyList_whenNoTasks` | Unitario | Lista vacía |
| 11 | `updateTask_shouldReturnUpdatedResponse_whenValid` | Unitario | TaskResponse con title y storyPoints actualizados |
| 12 | `updateTask_shouldThrowException_whenNotFound` | Unitario | `ResourceNotFoundException` con ID 777 |
| 13 | `delete_shouldCallDeleteById_whenTaskExists` | Unitario | `deleteById()` llamado una vez |
| 14 | `delete_shouldThrowException_whenTaskNotFound` | Unitario | `ResourceNotFoundException`, `deleteById()` nunca llamado |
| 15 | `findByStatus_shouldReturnFilteredList` | Unitario | Lista con una tarea en status TODO |

### 3.2 TaskControllerTest (14 tests de integración — @WebMvcTest + MockMvc)

| # | Nombre del test | Método HTTP | Endpoint | HTTP status esperado |
|---|-----------------|-------------|----------|----------------------|
| 1 | `createTask_shouldReturn201_whenValidRequest` | POST | `/api/tasks` | 201 Created |
| 2 | `createTask_shouldReturn400_whenTitleIsBlank` | POST | `/api/tasks` | 400 Bad Request |
| 3 | `createTask_shouldReturn400_whenStatusIsNull` | POST | `/api/tasks` | 400 Bad Request |
| 4 | `getAllTasks_shouldReturn200_withList` | GET | `/api/tasks` | 200 OK (lista de 2) |
| 5 | `getAllTasks_shouldReturn200_withEmptyList` | GET | `/api/tasks` | 200 OK (lista vacía) |
| 6 | `getTaskById_shouldReturn200_whenExists` | GET | `/api/tasks/{id}` | 200 OK |
| 7 | `getTaskById_shouldReturn404_whenNotFound` | GET | `/api/tasks/{id}` | 404 Not Found |
| 8 | `updateStatus_shouldReturn200_withUpdatedStatus` | PATCH | `/api/tasks/{id}/status` | 200 OK |
| 9 | `updateStatus_shouldReturn404_whenTaskNotFound` | PATCH | `/api/tasks/{id}/status` | 404 Not Found |
| 10 | `updateStatus_shouldReturn400_whenStatusIsNull` | PATCH | `/api/tasks/{id}/status` | 400 Bad Request |
| 11 | `deleteTask_shouldReturn204_whenExists` | DELETE | `/api/tasks/{id}` | 204 No Content |
| 12 | `deleteTask_shouldReturn404_whenNotFound` | DELETE | `/api/tasks/{id}` | 404 Not Found |
| 13 | `getByStatus_shouldReturn200_withFilteredTasks` | GET | `/api/tasks/status/{status}` | 200 OK (lista filtrada) |
| 14 | `updateTask_shouldReturn200_whenValid` | PUT | `/api/tasks/{id}` | 200 OK |

### 3.3 MiniJiraApplicationTest (1 test de smoke)

| # | Nombre del test | Tipo | Resultado esperado |
|---|-----------------|------|--------------------|
| 1 | `contextLoads` | Smoke / Context | El contexto Spring Boot levanta sin errores usando H2 en memoria |

---

## 4. Cobertura estimada por módulo

| Módulo | Cobertura estimada | Observación |
|--------|--------------------|-------------|
| `TaskController` | ~90% | Todos los endpoints cubiertos; happy path + errores |
| `TaskService` | ~85-90% | CRUD completo + casos de error; falta cobertura de edge cases en buildTaskFromRequest |
| `TaskRepository` | ~60% | Cubierto indirectamente por tests de integración |
| `UserController` | ~0% | Sin tests dedicados |
| `UserService` | ~0% | Sin tests dedicados |
| `ProjectController` | ~0% | Sin tests dedicados |
| `ProjectService` | ~0% | Sin tests dedicados |
| `GlobalExceptionHandler` | ~70% | Cubierto por tests del controller (400, 404) |
| **Global estimado** | **~40–45%** | Foco total en módulo Task |

---

## 5. Observaciones y recomendaciones

### Pendiente de alta prioridad

1. **Tests para UserService y ProjectService.** Las reglas de negocio (email único en UserService) no están cubiertas. Riesgo: regresiones silenciosas.
2. **Tests para UserController y ProjectController.** Los 6 endpoints de usuarios y proyectos no tienen ni un test de integración.

### Pendiente de media prioridad

3. **Test de edge case en TaskService.** El método `buildTaskFromRequest` con `projectId` inexistente no tiene test dedicado.
4. **Cobertura de `GlobalExceptionHandler` para 409 y 500.** El caso `BusinessException` (email duplicado) y el handler genérico `Exception` no tienen tests.

### Buenas prácticas observadas

- Uso correcto de `@BeforeEach` para fixture compartido.
- `@DisplayName` en todos los tests facilita lectura en reportes CI.
- Tests de controller verifican respuesta JSON con `jsonPath`, no solo status code.
- Tests de service verifican interacciones con `verify()` además del resultado.
- `ObjectMapper` configurado con `JavaTimeModule` para soportar `LocalDate`.

---

## 6. Entorno de test

| Componente | Versión / Configuración |
|------------|------------------------|
| JUnit | 5 (via spring-boot-starter-test) |
| Mockito | Via spring-boot-starter-test |
| MockMvc | @WebMvcTest slice |
| Base de datos (tests) | H2 en memoria (`jdbc:h2:mem:testdb`) |
| Dialect Hibernate (tests) | `H2Dialect` |
| DDL strategy (tests) | `create-drop` |
| Build tool | Maven 3.x |
| Java | 17 |
| Ejecución en sandbox | No disponible (sin mvn/java en entorno) — validación estructural |
