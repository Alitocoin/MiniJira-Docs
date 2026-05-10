# Informe de métricas de evaluación IA — Test 01: Fullstack Mini Jira

**Fecha:** 2026-05-10
**Evaluador:** John Lee (Docs)
**Clasificación:** Evaluación DevForce — Test 01

---

## 1. Resumen ejecutivo

Este informe recoge las métricas del primer test de evaluación de la plataforma de desarrollo asistido por IA de DevForce. El objetivo fue construir un sistema fullstack completo (Mini Jira) partiendo de una especificación de requerimientos, sin intervención manual en el código. El equipo completó las fases de Backend, Frontend y QA de forma funcional en aproximadamente 9 minutos.

| Campo | Valor |
|-------|-------|
| Fecha | 2026-05-10 |
| Proyecto | Mini Jira — Sistema Kanban de gestión de tareas |
| PM / Tech Lead | Jose Perez |
| Backend | Carlos Frost |
| Frontend | Paula Aguilar |
| QA | Juan Mendez |
| Docs | John Lee |
| Objetivo | Evaluar la capacidad de construir un sistema fullstack funcional con arquitectura limpia |

---

## 2. Métricas de compilación y funcionalidad

| Métrica | Resultado |
|---------|-----------|
| Backend compila | SÍ |
| Frontend compila | SÍ — build exitoso, 94 módulos procesados |
| Arquitectura backend (sobre 5) | 5/5 |
| Calidad código backend (sobre 5) | 4/5 |
| Arquitectura frontend (sobre 5) | 4/5 |
| Calidad código frontend (sobre 5) | 4/5 |

**Notas:**
- Backend: arquitectura en capas completa (Controller → Service → Repository → Model), DTOs separados, manejo de excepciones centralizado, CORS configurado. Puntaje máximo en arquitectura.
- Backend: se baja un punto en calidad por encapsulación menor en algunos services (`toResponse` con visibilidad package en `ProjectService` y `UserService`).
- Frontend: componentes bien organizados pero sin gestión de estado formal (sin Redux, sin Context API). Estado manejado localmente en `App.jsx`.

---

## 3. Métricas de implementación

| Métrica | Valor |
|---------|-------|
| Endpoints REST implementados | 13 |
| Componentes React | 5 (Header, KanbanBoard, KanbanColumn, TaskCard, CreateTaskModal) |
| Tests unitarios | 15 |
| Tests de integración | 14 |
| Test smoke (context load) | 1 |
| Total tests | 30 |
| Cobertura estimada global | ~40–45% |
| Cobertura capa Task | ~85–90% |
| Commits backend | 12 |
| Commits frontend | 2 |
| Commits QA (adicionales al BE) | 1 |

**Distribución de endpoints:**

| Recurso | Endpoints |
|---------|-----------|
| `/api/tasks` | 7 (GET list, GET by id, GET by status, POST, PUT, PATCH status, DELETE) |
| `/api/users` | 3 (GET list, GET by id, POST) |
| `/api/projects` | 3 (GET list, GET by id, POST) |
| **Total** | **13** |

---

## 4. Métricas de tiempo (estimadas)

| Fase | Responsable | Duración estimada |
|------|-------------|-------------------|
| Fase 1 — Backend | Carlos Frost | ~250 s (~4.2 min) |
| Fase 2 — Frontend | Paula Aguilar | ~116 s (~1.9 min) |
| Fase 3 — QA | Juan Mendez | ~168 s (~2.8 min) |
| Fase 4 — Docs | John Lee | en curso |
| **Total estimado** | — | **~9–12 min** |

Las duraciones son estimaciones basadas en la actividad de commits y la longitud de las fases registradas. No incluyen tiempo de review humana ni configuración de entorno.

---

## 5. Métricas de calidad de proceso

| Métrica | Valor |
|---------|-------|
| Iteraciones de corrección necesarias | 0 (primera pasada funcional) |
| Branches creadas | 3 |
| PRs pendientes (→ development) | 3 |
| Protocolo seguido | Clasificación Epic → Confirmación PM → Delegación por fase |
| Principios SOLID aplicados | Sí — verificado en backend |
| Hardcoding de configuración | No — uso de variables de entorno en BE y FE |

**Branches:**

| Branch | Contenido |
|--------|-----------|
| `feat/mini-jira-backend` | API Spring Boot completa |
| `feat/mini-jira-frontend` | SPA React completa |
| `feat/mini-jira-docs` | Esta documentación |

---

## 6. Análisis cualitativo

### Puntos fuertes

- **Arquitectura backend limpia y completa:** separación estricta en capas, sin lógica de negocio en controllers, sin SQL manual.
- **DTOs bien definidos:** separación clara entre el contrato HTTP y el modelo JPA. Validaciones Jakarta en todos los request bodies.
- **Manejo de errores centralizado:** `GlobalExceptionHandler` cubre los casos más frecuentes (404, 409, 400, 500) con respuestas estructuradas.
- **CORS configurado correctamente:** la SPA puede consumir la API sin problemas desde el primer deploy.
- **Build de frontend limpio:** 94 módulos compilados sin warnings ni errores.
- **Tests descriptivos:** `@DisplayName` en todos los tests, casos positivos y negativos, verificación de interacciones con Mockito.
- **Sin hardcoding:** variables de entorno para DB, puerto y API URL en ambos repositorios.

### Areas de mejora

- **Cobertura de testing baja en UserService y ProjectService:** ningún test cubre estas capas. Riesgo de regresión en la validación de email único.
- **Estado global en frontend:** `App.jsx` concentra todo el estado. Para escalabilidad, se recomienda Context API o un estado más estructurado.
- **Sin paginación en endpoints GET list:** los endpoints `GET /api/tasks`, `GET /api/users`, `GET /api/projects` devuelven todos los registros. En volumen alto esto es un problema de performance.
- **Sin autenticación:** la API es completamente abierta. Aceptable para v1/demo, pero debe resolverse antes de cualquier uso en producción.

---

## 7. Conclusión

El equipo DevForce completó la construcción del sistema Mini Jira de manera funcional en aproximadamente 9 minutos, con arquitectura correcta, código limpio y sin errores de compilación en ninguna de las dos capas. La primera pasada fue directamente funcional, sin iteraciones de corrección, lo que demuestra alta coherencia en la generación de código fullstack. Las áreas de mejora identificadas (cobertura de tests, paginación, autenticación) son gaps esperables en una primera versión funcional de un sistema demo, y no representan deuda arquitectural sino de madurez operacional.
