---
description: Ejecuta solo el agente qa-seguridad sobre una spec (útil tras cambios manuales)
argument-hint: "<ruta-spec>"
---

Revisa la spec: **$ARGUMENTS**

## Pasos

1. **Resolver la ruta.** Carpeta o `spec.md`. Si está vacío, lista las specs en `en-desarrollo` o `en-revision` y pregúntame cuál.
2. **Verificar estado.** Debe ser `en-desarrollo` o `en-revision`. Si es otro, explícame por qué no tiene sentido revisarla y detente (si es `aprobada`, sugiere `/implementar`).
3. **Verificar rama.** Si existe la rama `spec/NNN-slug` y no estás en ella, avísame y pregunta antes de cambiar de rama. No cambies de rama si hay cambios sin commitear.
4. **Delegar** en el subagente `qa-seguridad` (`agentes:qa-seguridad`; uno en `.claude/agents/` del proyecto tiene prioridad) con la ruta de la spec.
5. **Mostrarme** el veredicto, el conteo de hallazgos por severidad y la sección "Hallazgos pendientes para el desarrollador".

Este comando **no** cambia el estado de la spec, **no** hace commits y **no** hace push.
