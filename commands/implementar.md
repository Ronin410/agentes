---
description: Implementa una spec aprobada - desarrollador y luego qa-seguridad, con máximo 2 rondas; si QA aprueba, hace push de la rama spec/NNN-slug
argument-hint: "<ruta-spec>"
---

Implementa la spec: **$ARGUMENTS**

## Reglas

- **Nunca** hagas commit ni push a `main` (ni `master`). Todo ocurre en la rama `spec/NNN-slug`.
- Nunca uses `git push --force`.
- Máximo **2 rondas** de desarrollo + QA. Si QA rechaza dos veces seguidas, te detienes.

## Pasos

1. **Resolver la ruta.** El argumento puede ser la carpeta o el `spec.md`. Si está vacío, lista las specs en `aprobada` o `en-desarrollo` y pregúntame cuál. Obtén `NNN` y `slug` del nombre de la carpeta.
2. **Verificar estado.** Lee el frontmatter de `spec.md`:
   - `aprobada` o `en-desarrollo` → continúa.
   - `planeada` → **detente** y dime: "La spec está `planeada` pero no aprobada. Revisa `plan.md` y `tasks.md` y cambia el estado a `aprobada` para continuar." No implementes nada.
   - Cualquier otro → explica por qué no aplica y detente.
   - Verifica también que existan `plan.md` y `tasks.md`.
3. **Ronda 1 – desarrollo.** Delega en el subagente `desarrollador` (`agentes:desarrollador`; uno en `.claude/agents/` del proyecto tiene prioridad) con la ruta de la spec. Debe trabajar en la rama `spec/NNN-slug`, con un commit por tarea `spec NNN: <tarea>`.
4. **Ronda 1 – QA.** Delega en el subagente `qa-seguridad` (`agentes:qa-seguridad`) con la ruta de la spec. Debe escribir `specs/NNN-slug/review.md`.
5. **Leer el veredicto** de `review.md` (línea `**Veredicto:**`).
   - **APROBADO** → ve al paso 7.
   - **RECHAZADO** → ve al paso 6.
6. **Ronda 2** (solo una vez):
   1. Delega en `desarrollador` indicándole que existe `review.md` RECHAZADO y que atienda **solo** los hallazgos pendientes listados.
   2. Delega de nuevo en `qa-seguridad` (ronda 2).
   3. Si el veredicto es **APROBADO** → paso 7.
   4. Si vuelve a ser **RECHAZADO** → **detente**. No hagas push. Repórtame: rama, commits hechos, y la sección "Hallazgos pendientes para el desarrollador" de `review.md`. Deja el estado en `en-desarrollo`.
7. **Cerrar (APROBADO).**
   1. Confirma con `git branch --show-current` que estás en `spec/NNN-slug` y no en `main`.
   2. Cambia `estado` a `en-revision` en `spec.md`, agrega `review.md` y haz commit: `spec NNN: review aprobado, estado en-revision`.
   3. Haz push: `git push -u origin spec/NNN-slug`. Si falla por red, reintenta hasta 4 veces con espera creciente (2s, 4s, 8s, 16s).
   4. Repórtame: rama, lista de commits (`git log --oneline main..HEAD`), resumen de `review.md`, y el siguiente paso: **"Abre el PR de `spec/NNN-slug` hacia `main` y revisa el preview de Render. Cuando lo mergees, cambia el estado a `terminada`."**

No abras el PR tú mismo: eso lo hago yo.
