# ADR-003 — Stack frontend: React 18 + TypeScript strict + Vite

**Estado:** Aceptado
**Fecha:** 2026-05-11
**Autores:** Paula Aguilar (Frontend), Luis Barrios (Docs)
**Relacionado con:** Test 01 (fullstack base) — branch `feat/test-01-fullstack @ MiniJira-FE`

---

## Contexto

Al iniciar el proyecto MiniJira necesitábamos elegir el stack para el tablero Kanban en el navegador. Los criterios principales:

1. **Velocidad de desarrollo** — el equipo de frontend era de una sola persona (Paula Aguilar).
2. **Tipado** — el backend expone un contrato REST con tipos definidos; queríamos que el frontend validara ese contrato en tiempo de compilación.
3. **Ecosistema de testing** — necesitábamos poder probar componentes y funciones de API sin levantar un servidor real.
4. **Build** — el artefacto final se sirve desde Cloud Run o un CDN; el tamaño del bundle y la velocidad de build importan.

Las alternativas evaluadas:

| Opción | Pros | Contras |
|--------|------|---------|
| React + JavaScript | Menos setup | Sin seguridad de tipos; errores de contrato en runtime |
| React + TypeScript + Webpack | Maduro | Configuración lenta, HMR lento en desarrollo |
| **React + TypeScript + Vite** | Build rápido, HMR instantáneo, config mínima | — |
| Vue 3 + TypeScript | Alternativa válida | El equipo tenía más experiencia en React |
| Angular | TypeScript nativo | Overhead de framework para este tamaño de proyecto |

---

## Decisión

Se adoptó **React 18 + TypeScript (modo strict) + Vite** con las siguientes convenciones:

- **Estado centralizado en `App.tsx`** — un único punto de verdad para `tasks`, `users`, `error` y `loading`. Los componentes hijos reciben datos y callbacks por props. No se incorporó Redux ni Zustand dado el tamaño acotado del proyecto.
- **Cliente HTTP único en `api.ts`** — todas las llamadas al backend pasan por funciones exportadas desde este módulo. Los componentes nunca usan axios directamente. Esto simplifica el mocking en tests.
- **TypeScript strict** — `strict: true` en `tsconfig.json`. El check `tsc --noEmit` corre en el pipeline CI. Un build sin errores de TypeScript es condición necesaria para merge.
- **Testing** — Vitest + Testing Library. Vitest corre en el mismo entorno de Vite, sin configuración adicional de transpilación.

### Estructura de directorios relevante

```
src/
├── api.ts              cliente HTTP (axios.create + funciones tipadas)
├── types.ts            interfaces TypeScript: Task, User, Project, TaskStatus
├── App.tsx             estado global + callbacks + composición de la UI
├── components/
│   ├── BoardColumn.tsx
│   ├── TaskCard.tsx
│   ├── CreateTaskForm.tsx
│   ├── LoginForm.tsx
│   └── RegisterForm.tsx
└── test/
    ├── setup.ts
    ├── BoardColumn.test.tsx
    └── api.test.ts
```

---

## Consecuencias

**Positivas:**
- Vite reduce el tiempo de arranque en desarrollo a menos de 1 segundo. El HMR es instantáneo.
- TypeScript strict detectó incompatibilidades de tipos entre los DTOs del backend y el frontend durante el desarrollo, antes de llegar a runtime.
- El módulo `api.ts` como punto de entrada único facilitó escribir los 11 tests de `api.test.ts` mockeando únicamente axios; ningún componente necesita saber cómo hacer fetch.
- `tsc --noEmit` en CI actúa como segunda barrera de calidad después de los tests.

**Negativas:**
- El acoplamiento al estado de `App.tsx` crecerá a medida que el proyecto escale. Si el tablero incorpora más vistas (proyectos, usuarios, reportes), se recomienda evaluar un estado global más estructurado (Zustand o React Query).
- Sin tests para `CreateTaskForm`, `TaskCard`, `LoginForm` y `RegisterForm`. Ver gaps en QA Test Plan.

**Neutral:**
- La elección de Vite implica que los workers de build del CI usan `npm run build` (no webpack). Compatible con la configuración de GitHub Actions existente.
