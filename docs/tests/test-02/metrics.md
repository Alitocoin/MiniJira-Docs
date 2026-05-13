# Metricas de evaluacion — Test 02 Bug Fixing

Fecha de evaluacion: 2026-05-10

---

## Tabla de metricas

| Metrica                   | Resultado | Detalle                                                                 |
|---------------------------|-----------|-------------------------------------------------------------------------|
| Bug identificado          | Si        | Error de estado persistente en `CreateTaskForm` al reenviar formulario. |
| Bug corregido             | Si        | `setError(null)` agregado al inicio de `handleSubmit`.                  |
| No introduce nuevos errores | Si      | Build y flujos del formulario verificados post-fix.                     |
| Calidad de codigo (1–5)   | 5         | Fix minimo e idiomatico en React. Sin over-engineering. Sin efectos secundarios. |
| Tiempo de implementacion  | ~25 min   | Desde identificacion del bug hasta entrega del fix verificado.          |
| Numero de iteraciones     | 1         | Fix correcto en la primera iteracion.                                   |

---

## Archivos entregados

### Frontend — 1 archivo modificado

```
MiniJira-FE/
└── src/
    └── components/
        └── CreateTaskForm.tsx    ← setError(null) en handleSubmit
```

---

## Decisiones tecnicas destacadas

1. **Fix en el punto exacto**: se agrego `setError(null)` al inicio del handler en lugar de en el bloque `finally`. La limpieza en `finally` limpiaría el error antes de que el usuario lo vea, lo que es incorrecto UX. Limpiar al inicio del nuevo intento es el patron correcto.

2. **Alcance minimo**: no se modificaron otros archivos ni se refactorizó el componente completo. Un bug fix debe ser quirúrgico para facilitar la revisión y reducir el riesgo de regresiones.

---

## Observaciones del evaluador

_(Dejar en blanco para completar durante la revisión.)_
