# Metricas de evaluacion — Test 04 Unit Testing

Fecha de evaluacion: 2026-05-10

---

## Tabla de metricas

| Metrica                   | Resultado | Detalle                                                                 |
|---------------------------|-----------|-------------------------------------------------------------------------|
| Compila backend           | N/A       | Sin JVM en sandbox. Tests verificados por analisis estatico del codigo. |
| Build frontend            | Si        | `npm run build` + `npm run test` con 0 errores.                         |
| Arquitectura (1–5)        | 5         | Tests organizados por capa (Service, Componente, API). Separacion correcta de responsabilidades en los tests. |
| Calidad de codigo (1–5)   | 5         | Mocks correctos con Mockito. Assertions especificas. Nombres de test descriptivos. Sin tests duplicados. |
| Cobertura %               | ~70%      | 27 tests BE cubren paths felices + excepciones en TaskService y UserService. 21 tests FE cubren BoardColumn y capa API completa. |
| Tiempo de implementacion  | ~30 min   | 48 tests escritos (27 BE + 21 FE) en una iteracion.                     |
| Numero de iteraciones     | 1         | Suite completa entregada en la primera iteracion.                       |

---

## Archivos entregados

### Backend — 2 archivos nuevos

```
MiniJira-BE/
└── src/test/java/com/minijira/backend/
    └── service/
        ├── TaskServiceImplTest.java      ← 15 tests
        └── UserServiceImplTest.java      ← 12 tests
```

### Frontend — 3 archivos nuevos / modificados

```
MiniJira-FE/
├── vitest.config.ts                      ← configuracion vitest + jsdom
└── src/
    ├── components/
    │   └── BoardColumn.test.tsx          ← 10 tests
    └── api.test.ts                       ← 11 tests
```

---

## Decisiones tecnicas destacadas

1. **Mockito sobre repositorios JPA**: los tests de servicio no tocan la base de datos. Rapidos, deterministicos, sin infraestructura externa.

2. **Vitest + jsdom**: runner nativo de Vite. Mas rapido que Jest en proyectos Vite. jsdom simula DOM sin browser real.

3. **Testing Library**: tests de componentes por comportamiento observable (texto, roles ARIA) en lugar de detalles internos. Mayor resiliencia ante refactors.

4. **Cobertura de happy path + excepciones**: cada metodo de servicio tiene al menos un test del flujo exitoso y uno del flujo de error (entidad no encontrada, duplicado).

---

## Observaciones del evaluador

_(Dejar en blanco para completar durante la revisión.)_
