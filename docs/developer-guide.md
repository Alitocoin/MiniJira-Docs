# Developer Guide — MiniJira

Guia para un desarrollador que se incorpora al proyecto. Objetivo: entender la estructura, las convenciones y el flujo de trabajo en menos de 20 minutos.

---

## Estructura del proyecto

### MiniJira-BE

```
MiniJira-BE/
├── src/
│   ├── main/
│   │   ├── java/com/minijira/backend/
│   │   │   ├── BackendApplication.java
│   │   │   ├── config/
│   │   │   │   ├── CorsConfig.java
│   │   │   │   └── SecurityConfig.java
│   │   │   ├── controller/
│   │   │   │   ├── AuthController.java
│   │   │   │   ├── TaskController.java
│   │   │   │   ├── UserController.java
│   │   │   │   └── ProjectController.java
│   │   │   ├── service/
│   │   │   │   ├── AuthService.java          # interface
│   │   │   │   ├── AuthServiceImpl.java      # implementacion
│   │   │   │   ├── TaskService.java
│   │   │   │   ├── TaskServiceImpl.java
│   │   │   │   ├── UserService.java
│   │   │   │   ├── UserServiceImpl.java
│   │   │   │   ├── ProjectService.java
│   │   │   │   └── ProjectServiceImpl.java
│   │   │   ├── repository/
│   │   │   │   ├── TaskRepository.java
│   │   │   │   ├── UserRepository.java
│   │   │   │   └── ProjectRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Task.java
│   │   │   │   ├── TaskStatus.java           # enum TODO | IN_PROGRESS | DONE
│   │   │   │   ├── User.java
│   │   │   │   └── Project.java
│   │   │   ├── dto/
│   │   │   │   ├── TaskRequest.java
│   │   │   │   ├── TaskResponse.java
│   │   │   │   ├── UserRequest.java
│   │   │   │   ├── UserResponse.java
│   │   │   │   ├── ProjectRequest.java
│   │   │   │   ├── ProjectResponse.java
│   │   │   │   ├── LoginRequest.java
│   │   │   │   ├── RegisterRequest.java
│   │   │   │   ├── AuthResponse.java
│   │   │   │   ├── StatusUpdateRequest.java
│   │   │   │   └── ErrorResponse.java
│   │   │   ├── exception/
│   │   │   │   ├── GlobalExceptionHandler.java
│   │   │   │   ├── ResourceNotFoundException.java
│   │   │   │   └── BusinessException.java
│   │   │   └── security/
│   │   │       └── JwtUtil.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/minijira/backend/
│           ├── BackendApplicationTests.java
│           └── service/
│               ├── TaskServiceImplTest.java
│               └── UserServiceImplTest.java
└── pom.xml
```

### MiniJira-FE

```
MiniJira-FE/
├── src/
│   ├── App.tsx                  # estado global: session, tasks, loading, error
│   ├── main.tsx                 # monta React en el DOM
│   ├── api.ts                   # cliente Axios + todas las funciones HTTP
│   ├── types.ts                 # tipos: Task, User, Project, AuthSession, etc.
│   ├── components/
│   │   ├── BoardColumn.tsx      # columna del tablero Kanban
│   │   ├── TaskCard.tsx         # tarjeta individual
│   │   ├── CreateTaskForm.tsx   # modal de nueva tarea
│   │   ├── LoginForm.tsx        # formulario de autenticacion
│   │   └── RegisterForm.tsx     # formulario de registro
│   └── test/
│       ├── BoardColumn.test.tsx
│       ├── api.test.ts
│       └── setup.ts
├── dist/                        # generado por npm run build
├── package.json
└── vite.config.ts
```

---

## Convenciones de codigo

### Backend

**Arquitectura en capas**: `Controller -> Service (interface + impl) -> Repository`

Cada feature sigue este patron sin excepcion:

```
AuthController  ->  AuthService (interface)  ->  AuthServiceImpl  ->  UserRepository
TaskController  ->  TaskService (interface)  ->  TaskServiceImpl  ->  TaskRepository
```

El controller no accede al repository directamente. El service no conoce los detalles HTTP. Los repositorios son interfaces Spring Data JPA — no hay SQL manual salvo que sea estrictamente necesario.

**DTOs separados de entidades**

Las entidades JPA (`model/`) nunca salen del backend tal cual. Todo lo que entra o sale por HTTP pasa por DTOs:

- `*Request.java` — lo que el cliente envia (entrada)
- `*Response.java` — lo que el servidor devuelve (salida)
- La conversion entidad <-> DTO es responsabilidad del `ServiceImpl`

**Manejo de errores centralizado**

`GlobalExceptionHandler` (`@RestControllerAdvice`) captura:

| Excepcion | HTTP | Cuando usarla |
|---|---|---|
| `ResourceNotFoundException` | 404 | Entidad no encontrada por id |
| `BusinessException` | 409 | Violacion de regla de negocio |
| `MethodArgumentNotValidException` | 400 | Fallo de validacion `@Valid` |
| `Exception` (generico) | 500 | Cualquier error no previsto |

Todas las respuestas de error tienen la misma estructura (`ErrorResponse`): `status`, `error`, `message`, `path`.

**Seguridad**

- Spring Security filtra cada request antes de llegar al controller.
- Rutas publicas: `POST /api/auth/register` y `POST /api/auth/login`.
- El resto requiere `Authorization: Bearer <token>` en el header.
- `JwtUtil` genera y valida tokens HMAC-SHA con la clave `JWT_SECRET`.

**Swagger UI (solo desarrollo)**

Con el backend corriendo, la documentacion interactiva esta disponible en `http://localhost:8080/swagger-ui.html`. No requiere Postman ni ninguna herramienta externa. Para probar endpoints protegidos: hacer login via `POST /api/auth/login`, copiar el `token` de la respuesta, y pegarlo en el dialogo **Authorize** (candado en la esquina superior derecha). A partir de ahi todos los requests del navegador incluyen el header JWT automaticamente.

**Naming**

| Elemento | Convencion |
|---|---|
| Clases | PascalCase (`TaskServiceImpl`) |
| Metodos y variables | camelCase (`findById`, `taskList`) |
| Constantes | UPPER_SNAKE_CASE |
| Paquetes | lowercase (`com.minijira.backend.service`) |
| Tablas DB | gestionadas por Hibernate segun el nombre de la entidad |

**Validaciones**

Los campos obligatorios se anotan con `@NotBlank`, `@NotNull`, `@Size`, etc. en las clases `*Request`. Spring valida automaticamente cuando el controller usa `@Valid`. No repetir validaciones en el service.

---

### Frontend

**Componentes funcionales con TypeScript**

Todos los componentes son funciones React con tipos explícitos. No hay componentes de clase.

```ts
// Correcto
const BoardColumn: React.FC<BoardColumnProps> = ({ title, tasks, ... }) => { ... }

// Evitar
class BoardColumn extends React.Component { ... }
```

**Estado en App.tsx (single source of truth)**

El estado de la aplicacion vive en `App.tsx`: session de autenticacion, lista de tareas, loading, error. Los componentes hijos reciben datos y callbacks como props. No usar estado local para datos que otros componentes necesiten.

```ts
// App.tsx decide el estado
const [tasks, setTasks] = useState<Task[]>([]);
const [session, setSession] = useState<AuthSession | null>(null);
```

**Llamadas HTTP en api.ts**

Toda comunicacion con el backend pasa por `src/api.ts`. El cliente Axios esta configurado ahi con la base URL y el interceptor de JWT. Nunca hacer `fetch` o `axios.get` directamente en un componente.

```ts
// Correcto: importar desde api.ts
import { getTasks, createTask } from './api';

// Evitar: llamadas HTTP en componentes
const res = await fetch('http://localhost:8080/api/tasks');
```

La URL base del backend esta definida como constante en `api.ts`:
```ts
const BASE_URL = 'http://localhost:8080/api';
```
Para produccion, esta linea debe actualizarse antes del build (ver [docs/deployment-guide.md](deployment-guide.md)).

**Tipos en types.ts**

Todos los tipos y interfaces del dominio se definen en `src/types.ts`. Si agregás un campo nuevo a una entidad del backend, tambien actualizá su tipo aca.

**Autenticacion**

El token JWT se guarda en `localStorage` con las claves `auth_token`, `auth_username`, `auth_email`. El interceptor de Axios lo adjunta automaticamente a cada request. En logout se limpian las tres claves.

---

## Flujo de trabajo (gitflow simplificado)

```
main              <- produccion estable, solo merges desde development
  └── development <- integracion, rama base para features
        └── feat/test-XX  <- trabajo diario
```

**Reglas:**

1. Nunca commitear directamente a `main` ni a `development`.
2. Crear rama desde `development`: `git checkout -b feat/test-XX`.
3. PRs siempre hacia `development`, requieren revision de al menos un par.
4. Una vez validado en `development`, el PM coordina el merge a `main`.

**Commits — Conventional Commits en español**

```
feat: agregar endpoint GET /api/tags
fix: corregir validacion de email en RegisterRequest
refactor: extraer logica de mapeo a TaskMapper
chore: actualizar dependencia jjwt a 0.12.6
docs: agregar ejemplos de request en api-reference.md
test: cubrir caso de tarea no encontrada en TaskServiceImplTest
```

Formato: `<tipo>(<scope opcional>): <descripcion en minusculas, presente, imperativo>`

---

## Como agregar un endpoint nuevo al backend

Ejemplo: `GET /api/tags`

### Paso 1 — Crear la entidad

```java
// model/Tag.java
@Entity
@Table(name = "tags")
public class Tag {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String name;
    // getters y setters
}
```

### Paso 2 — Crear el repositorio

```java
// repository/TagRepository.java
public interface TagRepository extends JpaRepository<Tag, Long> { }
```

### Paso 3 — Crear el servicio (interface + impl)

```java
// service/TagService.java
public interface TagService {
    List<TagResponse> findAll();
}

// service/TagServiceImpl.java
@Service
public class TagServiceImpl implements TagService {
    private final TagRepository tagRepository;
    // constructor, implementacion
}
```

### Paso 4 — Crear el controller

```java
// controller/TagController.java
@RestController
@RequestMapping("/api/tags")
public class TagController {
    private final TagService tagService;

    @GetMapping
    public ResponseEntity<List<TagResponse>> getAll() {
        return ResponseEntity.ok(tagService.findAll());
    }
}
```

### Paso 5 — Agregar los DTOs

```java
// dto/TagRequest.java   <- campos de entrada con validaciones
// dto/TagResponse.java  <- campos de salida (sin datos sensibles)
```

### Paso 6 — Actualizar openapi.yaml

Agregar el path `/api/tags` con sus schemas de request/response y los codigos de respuesta posibles.

### Paso 7 — Escribir unit tests

```java
// test/.../service/TagServiceImplTest.java
@ExtendWith(MockitoExtension.class)
class TagServiceImplTest {
    @Mock TagRepository tagRepository;
    @InjectMocks TagServiceImpl tagService;

    @Test
    void findAll_returnsEmptyList_whenNoTags() { ... }
}
```

---

## Como agregar un componente nuevo al frontend

Ejemplo: `FilterBar` — barra de filtros por estado y asignado.

### Paso 1 — Definir tipos si aplica

Si el componente necesita props o datos nuevos, agregarlos en `src/types.ts`:

```ts
// types.ts
export interface FilterState {
  status: TaskStatus | 'ALL';
  assigneeId: number | null;
}
```

### Paso 2 — Crear el componente

```tsx
// src/components/FilterBar.tsx
import React from 'react';
import type { FilterState } from '../types';

interface FilterBarProps {
  value: FilterState;
  onChange: (filters: FilterState) => void;
}

const FilterBar: React.FC<FilterBarProps> = ({ value, onChange }) => {
  // implementacion
  return <div>...</div>;
};

export default FilterBar;
```

### Paso 3 — Agregar llamada API si aplica

Si el componente necesita datos del backend (ej. lista de usuarios para filtrar), agregar la funcion en `src/api.ts`:

```ts
export const getUsers = (): Promise<User[]> =>
  client.get<User[]>('/users').then((res) => res.data);
```

### Paso 4 — Importar en App.tsx

```tsx
// App.tsx
import FilterBar from './components/FilterBar';

// en el estado
const [filters, setFilters] = useState<FilterState>({ status: 'ALL', assigneeId: null });

// en el render
<FilterBar value={filters} onChange={setFilters} />
```

### Paso 5 — Escribir el test

```tsx
// src/test/FilterBar.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import FilterBar from '../components/FilterBar';

describe('FilterBar', () => {
  it('renderiza los controles de filtro', () => {
    render(<FilterBar value={{ status: 'ALL', assigneeId: null }} onChange={vi.fn()} />);
    expect(screen.getByRole('combobox')).toBeInTheDocument();
  });
});
```

---

## Preguntas frecuentes

**Los tests del backend fallan con error de datasource**
Los tests usan H2 en memoria. Verificar que existe `src/test/resources/application.properties` con la configuracion H2. No necesitan PostgreSQL.

**El frontend no conecta al backend**
La URL esta hardcodeada como `http://localhost:8080/api` en `src/api.ts`. Verificar que el backend esta corriendo en ese puerto y que CORS permite `http://localhost:5173`.

**El token JWT expira**
La expiracion por defecto es 86400000 ms (24 horas). Cambiar con la variable `JWT_EXPIRATION`. En desarrollo, hacer logout y login nuevamente.
