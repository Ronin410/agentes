---
description: Crea specs/NNN-slug/spec.md desde la plantilla, con número consecutivo y estado borrador
argument-hint: "<titulo>"
allowed-tools: Read, Glob, Bash(git config user.name), Bash(ls:*)
---

Crea una nueva spec con el título: **$ARGUMENTS**

## Pasos

1. Si el título está vacío, pídemelo y detente.
2. **Número**: lista las carpetas de `specs/` con formato `NNN-*` (tres dígitos). El nuevo número es el mayor + 1, con tres dígitos (`001` si no hay ninguna). Ignora `_plantilla.md` y carpetas que no sigan el formato.
3. **Slug**: a partir del título, en minúsculas, sin acentos ni `ñ` (usa `n`), espacios y símbolos convertidos a `-`, sin guiones repetidos ni al inicio/final, máximo 40 caracteres. Ejemplo: `login` → `login`; `Exportar a PDF` → `exportar-a-pdf`.
4. **Plantilla**: usa `specs/_plantilla.md`; si no existe, `${CLAUDE_PLUGIN_ROOT}/plantillas/spec.md`; si tampoco, `.claude/plantillas/spec.md`.
5. Crea `specs/NNN-slug/spec.md` con el frontmatter:
   - `id: NNN`
   - `titulo: <título tal cual>`
   - `estado: borrador`
   - `autor:` el nombre de `git config user.name` si existe; si no, déjalo vacío.
6. No llenes las secciones por mí (salvo el título); deja los textos guía de la plantilla.
7. Reporta la ruta creada y sugiere: "Llena la spec y luego corre `/planear specs/NNN-slug`".

Ejemplo: `/nueva-spec login` en un proyecto sin specs crea `specs/001-login/spec.md` con `estado: borrador`.
