---
description: Invoca al arquitecto para generar plan.md y tasks.md de una spec en borrador
argument-hint: "<ruta-spec>"
---

Planea la spec: **$ARGUMENTS**

## Pasos

1. **Resolver la ruta.** El argumento puede ser la carpeta (`specs/001-login`) o el archivo (`specs/001-login/spec.md`). Si está vacío, lista las specs en estado `borrador` y pregúntame cuál. Si no existe `spec.md` en esa carpeta, detente.
2. **Verificar estado.** Lee el frontmatter. Debe ser `borrador` (o `planeada` si te pido re-planear explícitamente). Si es otro, explícame por qué no se puede planear y detente.
3. **Delegar** en el subagente `arquitecto` (en el plugin aparece como `agentes:arquitecto`; si el proyecto tiene `.claude/agents/arquitecto.md`, ese tiene prioridad). Pásale la ruta de la carpeta de la spec y recuérdale que solo puede escribir dentro de `specs/`.
4. **Verificar el resultado**:
   1. Existen `plan.md` y `tasks.md` en la carpeta de la spec.
   2. El estado en `spec.md` es `planeada`.
   3. `git status --porcelain` no muestra archivos modificados fuera de `specs/` por este comando. Si los hay, avísame claramente.
5. **Mostrarme**:
   - Resumen del plan y número de tareas.
   - La sección `## Preguntas abiertas` completa (y `## Propuesta de división` si existe).
   - El mensaje: **"Para continuar, responde las preguntas abiertas (si las hay) y cambia `estado: planeada` → `estado: aprobada` en `spec.md`. Luego corre `/implementar <ruta-spec>`."**

No cambies el estado a `aprobada` tú mismo, aunque te lo sugiera el subagente.
