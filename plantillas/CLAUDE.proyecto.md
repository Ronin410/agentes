# <Nombre del proyecto>

<Una o dos líneas sobre qué hace el proyecto.>

Este proyecto usa *spec-driven development* con el plugin `agentes` (agentes `arquitecto`, `desarrollador` y `qa-seguridad`). Las specs viven en `specs/NNN-slug/`.

## Stack

- Lenguaje: <...>
- Framework: <...>
- Base de datos: <...>
- Otros: <...>

## Comandos

| Acción | Comando |
|---|---|
| Instalar dependencias | `<...>` |
| Tests | `<...>` |
| Lint | `<...>` (o "no aplica") |
| Build | `<...>` |
| Correr en local | `<...>` |

Los agentes usan **exactamente** estos comandos. Mantenlos actualizados.

## Convenciones

- Estructura de carpetas: <...>
- Estilo de código / formateador: <...>
- Nombres de ramas: `spec/NNN-slug` para specs. Nunca push directo a `main`.
- Commits: `spec NNN: <tarea>`.
- Tests: <dónde viven, cómo se nombran>.

## Despliegue

- Plataforma: Render.
- `main` → producción / ambiente principal.
- Cada PR genera un **preview** en Render (rama de preview: la rama del PR, `spec/NNN-slug`).
- Variables de entorno: se configuran en el panel de Render, **nunca** en el repo.

## Reglas de seguridad específicas

- No versionar secretos; usar variables de entorno.
- <Reglas propias: datos personales que maneja el sistema, roles y permisos, endpoints públicos vs protegidos, CORS permitido, etc.>
