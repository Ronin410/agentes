---
description: Prepara el proyecto actual para spec-driven development (CLAUDE.md, .claude/settings.json y specs/_plantilla.md)
argument-hint: "[stack opcional]"
allowed-tools: Read, Glob, Grep, Bash(git config user.name), Bash(ls:*)
---

Inicializa el proyecto actual para trabajar con los agentes `arquitecto`, `desarrollador` y `qa-seguridad`.

## Dónde están las plantillas

- Modo plugin (normal): `${CLAUDE_PLUGIN_ROOT}/plantillas/`
- Modo respaldo (GitHub Action): si la ruta anterior no existe o no se resolvió, usa `.claude/plantillas/` del proyecto.

Plantillas: `CLAUDE.proyecto.md`, `settings.proyecto.json`, `spec.md`.

## Pasos

1. Revisa qué existe ya en el proyecto: `CLAUDE.md`, `.claude/settings.json`, `specs/`. Detecta el stack mirando los archivos del repo (`package.json`, `*.csproj`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Gemfile`, etc.). Pista del usuario (opcional): $ARGUMENTS
2. **Pregúntame** (en un solo mensaje, proponiendo lo que detectaste como valor por defecto):
   1. Stack (lenguaje, framework, base de datos).
   2. Comando de instalación.
   3. Comando de tests.
   4. Comando de lint (si hay).
   5. Comando de build.
   6. Comando para correr en local.
   Espera mi respuesta antes de escribir archivos.
3. **`CLAUDE.md`**: si **no** existe, créalo desde `CLAUDE.proyecto.md` llenando las secciones con mis respuestas. Si ya existe, **no lo sobrescribas**: muéstrame qué secciones de la plantilla le faltan y ofrece agregarlas.
4. **`.claude/settings.json`**: si no existe, cópialo desde `settings.proyecto.json`. Si existe, agrega (fusionando el JSON, sin borrar nada) las claves `extraKnownMarketplaces.alejandro-agentes` y `enabledPlugins["agentes@alejandro-agentes"]`.
5. **`specs/_plantilla.md`**: crea la carpeta `specs/` si no existe y copia ahí `spec.md` como `_plantilla.md`.
6. Reporta qué archivos creaste o modificaste y sugiere el siguiente paso: `/nueva-spec <titulo>`.
