# QA Test Plan — MiniJira

**Versión:** 1.0.0
**Fecha:** 2026-05-11
**Autor:** Juan Mendez (QA) — documentado por Luis Barrios (Docs)
**Sistema:** MiniJira — gestión de tareas estilo Jira con tablero Kanban
**Stack:** Spring Boot 3 / Java 17 + PostgreSQL · React 18 / TypeScript + Vite

---

## 1. Resumen de estado actual

| Capa | Suite | Tests escritos | Estado | Cobertura medida |
|------|-------|---------------|--------|-----------------|
| Backend | `TaskServiceImplTest` | 13 | Escritos, pendientes de ejecución (sin JVM en sandbox) | No medida |
| Backend | `UserServiceImplTest` | 11 | Escritos, pendientes de ejecución (sin JVM en sandbox) | No medida |
| Frontend | `BoardColumn.test.tsx` | 10 | 10/10 verde | No medida |
| Frontend | `api.test.ts` | 11 | 11/11 verde | No medida |
| **Total** | | **45** | **21 verdes · 24 pendientes JVM** | |

---

## 2. Inventario de tests existentes

### 2.1 Backend — `TaskServiceImplTest.java`

Herramientas: JUnit 5 + Mockito + AssertJ. Mocks: `TaskRepository`, `UserRepository`, `ProjectRepository`.

| # | Nombre del test | Qué verifica |
|---|-----------------|--------------|
| 1 | `getAllTasks_returnsListOfTasks` | `findAll()` devuelve 2 tareas mapeadas a DTO con título y status correctos |
| 2 | `getAllTasks_emptyRepository_returnsEmptyList` | `findAll()` devuelve lista vacía cuando el repositorio no tiene registros |
| 3 | `getTaskById_existingId_returnsTask` | `findById(10L)` retorna DTO con id, título, status y storyPoints correctos |
| 4 | `getTaskById_nonExistingId_throwsResourceNotFoundException` | `findById(99L)` lanza `ResourceNotFoundException` con el id en el mensaje |
| 5 | `createTask_validRequest_returnsTaskResponse` | `create()` persiste tarea y aplica status `TODO` por defecto cuando no se especifica |
| 6 | `createTask_withExplicitStatus_respectsProvidedStatus` | `create()` respeta el status `IN_PROGRESS` cuando viene en el request |
| 7 | `createTask_withAssigneeAndProject_resolvesRelations` | `create()` resuelve relaciones `assignee` y `project` desde los repositorios correspondientes |
| 8 | `createTask_nonExistingAssigneeId_throwsResourceNotFoundException` | `create()` lanza excepción si el `assigneeId` no existe; nunca llama a `save()` |
| 9 | `createTask_nonExistingProjectId_throwsResourceNotFoundException` | `create()` lanza excepción si el `projectId` no existe; nunca llama a `save()` |
| 10 | `updateTaskStatus_validStatus_updatesCorrectly` | `updateStatus()` cambia el status a `IN_PROGRESS` y persiste el cambio |
| 11 | `updateTaskStatus_nonExistingId_throwsResourceNotFoundException` | `updateStatus(55L)` lanza excepción cuando la tarea no existe |
| 12 | `updateTaskStatus_toDone_setsCorrectFinalStatus` | `updateStatus()` acepta `DONE` como status final válido |
| 13 | `deleteTask_existingId_deletesSuccessfully` | `delete(1L)` llama a `existsById` y luego a `deleteById` |
| 14 | `deleteTask_nonExistingId_throwsResourceNotFoundException` | `delete(77L)` lanza excepción y nunca llama a `deleteById` |
| 15 | `createTask_noAssigneeNoProject_responseHasNullRelations` | DTO retornado tiene `assignee` y `project` nulos cuando no se proporcionan |

> Nota: el reporte de sesión indica 13 tests en esta suite. El archivo fuente contiene 15 métodos `@Test`. Se usa el conteo real del código.

### 2.2 Backend — `UserServiceImplTest.java`

Herramientas: JUnit 5 + Mockito + AssertJ. Mock: `UserRepository`.

| # | Nombre del test | Qué verifica |
|---|-----------------|--------------|
| 1 | `getAllUsers_returnsListOfUsers` | `findAll()` devuelve 2 usuarios mapeados a DTO con username y email correctos |
| 2 | `getAllUsers_emptyRepository_returnsEmptyList` | `findAll()` devuelve lista vacía |
| 3 | `getUserById_existingId_returnsUser` | `findById(10L)` retorna DTO con id, username, email y createdAt |
| 4 | `getUserById_nonExistingId_throwsResourceNotFoundException` | `findById(999L)` lanza `ResourceNotFoundException` con el id en el mensaje |
| 5 | `createUser_validRequest_returnsUserResponse` | `create()` persiste y devuelve DTO cuando username y email son únicos |
| 6 | `createUser_duplicateUsername_throwsBusinessException` | `create()` lanza `BusinessException` si el username ya existe; nunca llama a `save()` |
| 7 | `createUser_duplicateEmail_throwsBusinessException` | `create()` lanza `BusinessException` si el email ya existe; nunca llama a `save()` |
| 8 | `updateUser_validRequest_updatesAndReturnsDto` | `update()` persiste los nuevos username y email cuando no hay conflictos |
| 9 | `updateUser_sameUsernameAndEmail_doesNotThrowDuplicateException` | `update()` no falla cuando el usuario mantiene sus mismos username y email |
| 10 | `updateUser_nonExistingId_throwsResourceNotFoundException` | `update(42L)` lanza excepción cuando el usuario no existe |
| 11 | `deleteUser_existingId_deletesSuccessfully` | `delete(3L)` llama a `existsById` y luego a `deleteById` |
| 12 | `deleteUser_nonExistingId_throwsResourceNotFoundException` | `delete(404L)` lanza excepción y nunca llama a `deleteById` |

> Nota: el archivo fuente contiene 12 métodos `@Test`.

### 2.3 Frontend — `BoardColumn.test.tsx`

Herramientas: Vitest + Testing Library + userEvent. Componente bajo prueba: `BoardColumn`.

| # | Nombre del test | Qué verifica |
|---|-----------------|--------------|
| 1 | `renderiza el título de la columna` | El texto del prop `title` aparece en el DOM |
| 2 | `renderiza el badge con el conteo correcto de tareas` | El badge muestra el número de tareas pasadas |
| 3 | `renderiza las tareas pasadas como prop` | Los títulos de las tareas aparecen en el DOM |
| 4 | `renderiza exactamente la misma cantidad de TaskCards que tareas recibidas` | La cantidad de botones "Eliminar" iguala la cantidad de tareas |
| 5 | `muestra mensaje "Sin tareas" cuando no hay tareas` | Estado vacío muestra el texto "Sin tareas" |
| 6 | `no muestra el mensaje vacío cuando hay tareas` | "Sin tareas" no aparece cuando hay al menos una tarea |
| 7 | `llama a onDelete con el id correcto al hacer click en Eliminar` | Click en "Eliminar" llama al callback con el id de la tarea |
| 8 | `llama a onStatusChange con el id y status correcto al avanzar tarea` | Click en el botón "→" llama a `onStatusChange` con id y `'IN_PROGRESS'` |
| 9 | `renderiza columna IN_PROGRESS sin errores` | La columna con status `IN_PROGRESS` renderiza sin excepciones |
| 10 | `renderiza columna DONE sin errores y con badge correcto` | La columna `DONE` muestra el badge con el conteo correcto |

### 2.4 Frontend — `api.test.ts`

Herramientas: Vitest + mock de axios. Módulo bajo prueba: `api.ts`.

| # | Suite | Nombre del test | Qué verifica |
|---|-------|-----------------|--------------|
| 1 | `getTasks` | `hace GET a /tasks y devuelve la lista de tareas` | Llama a `GET /tasks`, retorna array tipado |
| 2 | `getTasks` | `devuelve lista vacía cuando el backend responde con []` | Retorna `[]` sin errores |
| 3 | `getTasks` | `propaga error de red cuando la petición falla` | Rechaza la promesa con `'Network Error'` |
| 4 | `updateTaskStatus` | `hace PATCH a /tasks/{id}/status con el status correcto` | Llama a `PATCH /tasks/5/status` con `{ status: 'IN_PROGRESS' }` |
| 5 | `updateTaskStatus` | `hace PATCH con status DONE correctamente` | Llama a `PATCH /tasks/10/status` con `{ status: 'DONE' }` |
| 6 | `updateTaskStatus` | `propaga error de red al actualizar status` | Rechaza con `'Connection refused'` |
| 7 | `updateTaskStatus` | `propaga error 404 cuando la tarea no existe` | Rechaza con objeto `{ response: { status: 404 } }` |
| 8 | `createTask` | `hace POST a /tasks con los datos correctos` | Llama a `POST /tasks` con el payload completo; retorna tarea creada |
| 9 | `deleteTask` | `hace DELETE a /tasks/{id} y resuelve void` | Llama a `DELETE /tasks/3`; retorna `undefined` |
| 10 | `deleteTask` | `propaga error 404 al intentar eliminar tarea inexistente` | Rechaza con `{ response: { status: 404 } }` |
| 11 | `getUsers` | `hace GET a /users y devuelve lista de usuarios` | Llama a `GET /users`; retorna array con username |

---

## 3. Cobertura actual vs objetivo

| Componente | Tests existentes | Cobertura estimada | Objetivo | Gap |
|------------|-----------------|-------------------|----------|-----|
| `TaskServiceImpl` | 15 tests unitarios | ~85% líneas (estimado) | 80% | Sin gap estimado; pendiente medir con JaCoCo |
| `UserServiceImpl` | 12 tests unitarios | ~85% líneas (estimado) | 80% | Sin gap estimado; pendiente medir con JaCoCo |
| `AuthServiceImpl` | 0 tests | 0% | 80% | Gap crítico |
| `ProjectServiceImpl` | 0 tests | 0% | 80% | Gap crítico |
| `AuthController` | 0 tests de integración | 0% | Prueba de humo | Gap alto |
| `BoardColumn` (FE) | 10 tests | ~90% ramas (estimado) | 80% | Sin gap estimado |
| `api.ts` (FE) | 11 tests | ~95% ramas (estimado) | 80% | Sin gap estimado |
| `CreateTaskForm` (FE) | 0 tests | 0% | 70% | Gap medio |
| `TaskCard` (FE) | 0 tests | 0% | 70% | Gap medio |
| `LoginForm` (FE) | 0 tests | 0% | 70% | Gap medio |
| `RegisterForm` (FE) | 0 tests | 0% | 70% | Gap medio |
| E2E (flujo completo) | 0 tests | 0% | 1 suite smoke | Gap alto |

> Cobertura estimada calculada a partir de los caminos ejercidos en los tests. Las cifras reales requieren JaCoCo (BE) y Vitest `--coverage` (FE).

---

## 4. Tests pendientes de implementar

### 4.1 Backend

**`AuthServiceImpl` — sin tests unitarios**
- `register()` con request válido devuelve `AuthResponse` con token
- `register()` con username duplicado lanza `BusinessException`
- `register()` con email duplicado lanza `BusinessException`
- `login()` con credenciales correctas devuelve JWT válido
- `login()` con password incorrecto lanza `BadCredentialsException`
- `login()` con usuario inexistente lanza `ResourceNotFoundException`

**`ProjectServiceImpl` — sin tests unitarios**
- `findAll()` devuelve lista de proyectos
- `findById()` existente y no existente
- `create()` con request válido
- `delete()` existente y no existente

**`AuthController` — sin tests de integración**
- `POST /api/auth/register` → 200 con token
- `POST /api/auth/register` → 409 con username duplicado
- `POST /api/auth/login` → 200 con token
- `POST /api/auth/login` → 401 con password incorrecto

### 4.2 Frontend

**`CreateTaskForm` — sin tests**
- Renderiza el formulario con los campos esperados
- Submit con datos válidos llama a `onSubmit` con el payload correcto
- Validación de campo requerido (título)
- Estado de carga mientras se procesa el submit

**`TaskCard` — sin tests**
- Renderiza el título y descripción de la tarea
- Muestra storyPoints y estimatedHours cuando no son null
- Muestra "—" u otro placeholder cuando son null

**`LoginForm` / `RegisterForm` — sin tests**
- Renderiza campos de email/password/username
- Submit con datos vacíos no llama al callback
- Submit con datos válidos llama a `onSubmit` con las credenciales

### 4.3 Tests E2E

No se implementó ningún test E2E. Se recomienda una suite smoke con Playwright que cubra:
1. Registro de usuario → login → obtener token
2. Crear tarea → verla en columna TODO del tablero
3. Mover tarea de TODO a IN_PROGRESS → verificar cambio en UI
4. Eliminar tarea → verificar que desaparece del tablero

---

## 5. Estrategia de testing recomendada

```
Nivel 1: Unitario (actual)
  Backend  → JUnit 5 + Mockito (servicios e implementaciones)
  Frontend → Vitest + Testing Library (componentes + api.ts)

Nivel 2: Integración (pendiente)
  Backend  → @SpringBootTest + MockMvc + H2 (controllers)
  Frontend → MSW (Mock Service Worker) para tests con fetch real

Nivel 3: E2E (pendiente)
  Playwright — flujos críticos: auth, CRUD de tareas, cambio de status
```

### Herramientas por capa

| Capa | Herramienta | Cobertura | Config requerida |
|------|-------------|-----------|-----------------|
| BE unitario | JUnit 5 + Mockito | Servicios | `mvn test` con Java 17 |
| BE cobertura | JaCoCo Maven Plugin | Líneas/ramas | Agregar a `pom.xml` |
| BE integración | `@SpringBootTest` + MockMvc | Controllers | Base H2 en `application-test.properties` |
| FE unitario | Vitest + Testing Library | Componentes | `npm test` ya configurado |
| FE cobertura | Vitest `--coverage` + c8 | Líneas/ramas | Agregar `--coverage` al script |
| E2E | Playwright | Flujos completos | Nuevo repo o directorio `e2e/` |

### Umbrales de cobertura recomendados

| Capa | Umbral mínimo | Justificación |
|------|--------------|---------------|
| Backend — servicios | 80% líneas | Capa de negocio crítica |
| Backend — controllers | 70% líneas (smoke) | Cubiertos principalmente por integración |
| Frontend — componentes | 70% ramas | UI cambia con frecuencia |
| Frontend — api.ts | 85% ramas | Contrato con el backend |

---

## 6. Riesgos de calidad abiertos

| ID | Riesgo | Impacto | Acción recomendada |
|----|--------|---------|-------------------|
| R01 | Tests de backend no ejecutados en CI real | Alto | Ejecutar `mvn test` con Java 17 antes de merge a `development` |
| R02 | Plugin Checkstyle puede no estar declarado en `pom.xml` | Medio | Verificar `pom.xml` y correr `mvn checkstyle:check` localmente |
| R03 | Sin cobertura formal medida en frontend | Bajo | Agregar `--coverage` a Vitest y configurar umbral mínimo (sugerido: 80%) |
| R04 | Auth JWT no protege endpoints CRUD existentes | Alto (producción) | Decisión intencional para test-03; proteger antes de merge a `main` |
| R05 | Branches test-02, test-03, test-04 sin pipeline CI/CD propio | Medio | Extender workflow de test-05 o crear workflows específicos por branch |

---

## 7. Orden de merge recomendado

Una vez que CI esté verde en cada branch, hacer merge en orden secuencial hacia `development`:

```
feat/test-01-fullstack
  → feat/test-02-bug-fixing
    → feat/test-03-user-feature
      → feat/test-04-unit-testing
        → feat/test-05-cicd
          → development
```

Resolver conflictos manualmente en cada merge dado que las branches partieron de `development` en distintos momentos.
