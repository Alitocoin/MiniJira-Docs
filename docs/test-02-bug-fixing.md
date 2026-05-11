# Test 02 — Bug Fixing: Error de estado al crear tarea

**Fecha:** 2026-05-10  
**Branch:** `feat/test-02-bug-fixing`  
**Estado:** Completado

---

## 1. Descripción del bug

Al crear una tarea desde el formulario del frontend, si la operación fallaba (error de red o validación), el mensaje de error aparecía correctamente. Sin embargo, al intentar enviar el formulario nuevamente con datos correctos, el error persistía en pantalla aunque la tarea se creara con éxito.

**Causa raíz:** la función `handleSubmit` en `CreateTaskForm.tsx` no limpiaba el estado de error (`setError(null)`) al inicio de cada nuevo intento de submit.

---

## 2. Archivos modificados

### Frontend

| Archivo | Cambio |
|---------|--------|
| `src/components/CreateTaskForm.tsx` | Agregar `setError(null)` al inicio de `handleSubmit` |

---

## 3. Cambio aplicado

**Antes:**
```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();
  try {
    await createTask(formData);
    onTaskCreated();
  } catch (err) {
    setError('Error al crear la tarea');
  }
};
```

**Después:**
```tsx
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();
  setError(null);          // ← fix: limpiar error previo
  try {
    await createTask(formData);
    onTaskCreated();
  } catch (err) {
    setError('Error al crear la tarea');
  }
};
```

---

## 4. Verificación

- [x] El error desaparece al iniciar un nuevo intento de submit
- [x] El mensaje de error sigue apareciendo correctamente cuando la operación falla
- [x] No se introdujeron regresiones en otros flujos del formulario
- [x] `npm run build` pasa sin errores

---

## 5. Métricas

Ver [`docs/test-02-metrics.md`](./test-02-metrics.md)

---

## 6. Decisiones técnicas

**Limpiar estado al inicio del handler**: el patrón correcto en React para formularios con feedback de error es resetear el estado de error al comienzo de cada nuevo intento, no en el `finally`. Esto garantiza que el usuario no vea errores fantasma de intentos anteriores aunque el nuevo submit esté en vuelo.
