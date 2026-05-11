# Test 04 — Unit Testing: Cobertura Backend y Frontend

**Fecha:** 2026-05-10  
**Branch:** `feat/test-04-unit-testing`  
**Estado:** Completado

---

## 1. Descripción

Implementación de suite completa de tests unitarios para las capas de servicio del backend (Spring Boot + JUnit 5 + Mockito) y para los componentes y funciones del frontend (React + Vitest + Testing Library).

**Objetivo:** Alcanzar cobertura funcional en las capas de lógica de negocio más críticas, documentar el comportamiento esperado del sistema y garantizar que los cambios futuros no rompen contratos existentes.

---

## 2. Archivos creados

### Backend

| Archivo | Tests | Capa |
|---------|-------|------|
| `TaskServiceImplTest.java` | 15 | Service |
| `UserServiceImplTest.java` | 12 | Service |

**Total backend**: 27 tests unitarios

### Frontend

| Archivo | Tests | Capa |
|---------|-------|------|
| `BoardColumn.test.tsx` | 10 | Componente |
| `api.test.ts` | 11 | API layer |
| `vitest.config.ts` | — | Configuración |

**Total frontend**: 21 tests unitarios

**Total sesión**: **48 tests** escritos en Test 04

---

## 3. Casos de prueba — Backend

### TaskServiceImplTest (15 tests)

| # | Caso | Método testeado |
|---|------|-----------------|
| 1 | Obtener todas las tareas devuelve lista | `getAllTasks()` |
| 2 | Obtener tarea por ID existente | `getTaskById(id)` |
| 3 | Obtener tarea por ID no existente lanza excepción | `getTaskById(id)` |
| 4 | Crear tarea con datos válidos | `createTask(dto)` |
| 5 | Crear tarea sin título lanza excepción | `createTask(dto)` |
| 6 | Actualizar tarea existente | `updateTask(id, dto)` |
| 7 | Actualizar tarea no existente lanza excepción | `updateTask(id, dto)` |
| 8 | Eliminar tarea existente | `deleteTask(id)` |
| 9 | Eliminar tarea no existente lanza excepción | `deleteTask(id)` |
| 10 | Cambiar estado a IN_PROGRESS | `updateStatus(id, status)` |
| 11 | Cambiar estado a DONE | `updateStatus(id, status)` |
| 12 | Cambiar estado a TODO | `updateStatus(id, status)` |
| 13 | Asignar usuario existente a tarea | `createTask(dto)` |
| 14 | Asignar usuario no existente lanza excepción | `createTask(dto)` |
| 15 | Asignar proyecto a tarea | `createTask(dto)` |

### UserServiceImplTest (12 tests)

| # | Caso | Método testeado |
|---|------|-----------------|
| 1 | Listar todos los usuarios | `getAllUsers()` |
| 2 | Obtener usuario por ID existente | `getUserById(id)` |
| 3 | Obtener usuario por ID no existente lanza excepción | `getUserById(id)` |
| 4 | Crear usuario con datos válidos | `createUser(dto)` |
| 5 | Crear usuario con username duplicado lanza excepción | `createUser(dto)` |
| 6 | Crear usuario con email duplicado lanza excepción | `createUser(dto)` |
| 7 | Actualizar usuario existente | `updateUser(id, dto)` |
| 8 | Actualizar usuario no existente lanza excepción | `updateUser(id, dto)` |
| 9 | Eliminar usuario existente | `deleteUser(id)` |
| 10 | Eliminar usuario no existente lanza excepción | `deleteUser(id)` |
| 11 | Buscar usuario por email existente | `findByEmail(email)` |
| 12 | Buscar usuario por email no existente | `findByEmail(email)` |

---

## 4. Casos de prueba — Frontend

### BoardColumn.test.tsx (10 tests)

| # | Caso |
|---|------|
| 1 | Renderiza el título de la columna |
| 2 | Renderiza las tareas pasadas como prop |
| 3 | Muestra estado vacío cuando no hay tareas |
| 4 | Llama `onStatusChange` al hacer click en botón de avance |
| 5 | Llama `onDelete` al hacer click en botón de eliminar |
| 6 | No renderiza botón de avance en columna DONE |
| 7 | Muestra el assignee de la tarea si existe |
| 8 | No muestra assignee si la tarea no tiene asignado |
| 9 | Aplica clase CSS correcta según el status |
| 10 | Renderiza múltiples tareas en la misma columna |

### api.test.ts (11 tests)

| # | Caso |
|---|------|
| 1 | `getTasks()` llama GET /api/tasks |
| 2 | `getTasks()` retorna el array de tareas |
| 3 | `createTask()` llama POST /api/tasks con body correcto |
| 4 | `updateTaskStatus()` llama PATCH /api/tasks/{id}/status |
| 5 | `deleteTask()` llama DELETE /api/tasks/{id} |
| 6 | `getUsers()` llama GET /api/users |
| 7 | `login()` llama POST /api/auth/login |
| 8 | `login()` retorna AuthResponse con token |
| 9 | `register()` llama POST /api/auth/register |
| 10 | Interceptor adjunta Bearer token cuando existe en localStorage |
| 11 | Interceptor no adjunta token cuando localStorage está vacío |

---

## 5. Configuración de testing

### Backend (`pom.xml`)

- `spring-boot-starter-test` (JUnit 5 + Mockito integrado)
- `@ExtendWith(MockitoExtension.class)` en cada clase de test
- `@Mock` para repositorios, `@InjectMocks` para servicios

### Frontend (`vitest.config.ts`)

```ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/setupTests.ts'],
    globals: true,
  },
});
```

---

## 6. Métricas

Ver [./metrics.md](./metrics.md)

---

## 7. Decisiones técnicas

**Mockito para servicios**: los tests unitarios mockean los repositorios JPA, aislando la lógica del servicio de la base de datos. Esto permite correr 27 tests en milisegundos sin infraestructura.

**Vitest + jsdom**: Vitest es el runner nativo de Vite, más rápido que Jest en proyectos Vite. jsdom simula el DOM del browser sin levantar un browser real.

**Testing Library**: los tests de componentes interactúan con el DOM como lo haría un usuario (por texto visible, roles ARIA) en lugar de por detalles de implementación (clases CSS, estructura interna). Esto hace los tests más robustos ante refactors de UI.
