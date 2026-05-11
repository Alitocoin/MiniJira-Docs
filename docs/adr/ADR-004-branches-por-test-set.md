# ADR-004 — Estrategia de branches: una branch por test set desde development

**Estado:** Aceptado
**Fecha:** 2026-05-11
**Autores:** Jose Perez (PM), equipo DevForce, Luis Barrios (Docs)
**Relacionado con:** todos los tests (01 al 05) en los 3 repos

---

## Contexto

El proyecto MiniJira se ejecutó como una suite de 5 test sets con specialists distintos por área (Carlos Frost en backend, Paula Aguilar en frontend, Juan Mendez en QA, John Lee en CI/CD). Cada test set tenía un alcance delimitado y podía desarrollarse en paralelo o en secuencia. Necesitábamos una estrategia de branches que:

1. Garantizara trazabilidad entre el trabajo entregado y el requerimiento que lo originó.
2. Permitiera revisar el código de cada test set de forma independiente.
3. Fuera compatible con el flujo de merge hacia `development` y eventualmente hacia `main`.

Las opciones consideradas:

- **Branch única `development`** — todos los cambios directo a development. Simple pero sin trazabilidad por test; dificulta revisión y rollback aislado.
- **Una branch por specialist** — branches por persona, no por requerimiento. Dificulta entender qué branch corresponde a qué funcionalidad.
- **Una branch por test set** — `feat/test-01-*`, `feat/test-02-*`, etc. Trazabilidad directa entre branch y criterio de aceptación.

---

## Decisión

Se adoptó **una branch `feat/test-XX-<descripción>` por test set**, creada desde `development`, aplicada en los 3 repos (`MiniJira-BE`, `MiniJira-FE`, `MiniJira-Docs`).

### Convención de naming

```
feat/test-01-fullstack         Test 01 — fullstack base (BE + FE + Docs)
feat/test-02-bug-fixing        Test 02 — bug fix tablero Kanban (FE)
feat/test-03-user-feature      Test 03 — autenticación JWT (BE + FE)
feat/test-04-unit-testing      Test 04 — unit tests (BE + FE)
feat/test-05-cicd              Test 05 — pipeline CI/CD (BE + FE + Docs)
```

### Orden de merge

Los merges hacia `development` se realizan en orden secuencial (01 → 02 → 03 → 04 → 05) dado que cada test set asume que el anterior está integrado.

### Trazabilidad por commit

Cada commit en las branches usa el formato Conventional Commits en español, permitiendo identificar qué cambio corresponde a qué test. Los SHAs se documentan en el reporte de sesión `REPORT-2026-05-11.md`.

---

## Consecuencias

**Positivas:**
- Trazabilidad directa: dado un bug en producción, se puede identificar en qué test set se introdujo el cambio.
- Las branches quedan como referencia histórica del trabajo entregado en cada etapa.
- Permite hacer code review por test set antes del merge.
- Compatible con el proceso PMOS (Project Management Operating System) del equipo que organiza el trabajo por sesiones y test sets.

**Negativas:**
- Las branches partieron de `development` en distintos momentos de la sesión. Al hacer merge secuencial pueden surgir conflictos que deben resolverse manualmente, especialmente entre test-01 y test-03 (ambos tocaron `App.tsx` y `api.ts` en frontend).
- Si se agregan más features después, los branches de test quedan como ramas huérfanas en el historial. Se recomienda eliminarlas después del merge.

**Neutral:**
- La branch de `MiniJira-Docs` sigue el mismo patrón aunque la documentación no tiene "código" que entre en conflicto. Facilita la revisión de documentación junto con el código que documenta.
- La branch `improvement/docs-completa` (esta branch) sigue el mismo patrón para la Fase 3 de documentación (QA Test Plan, CHANGELOG, ADRs).
