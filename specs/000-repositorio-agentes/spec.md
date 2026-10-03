---
id: 000
titulo: Repositorio claude-agentes (plugin de agentes por roles)
estado: aprobada
autor: Alejandro
---

# Spec 000: Repositorio `claude-agentes`

## Objetivo

Crear un repositorio de GitHub llamado `claude-agentes` que funcione como **plugin de Claude Code** y contenga un equipo estandarizado de agentes por roles (Arquitecto, Desarrollador, QA/Seguridad), comandos y plantillas. Debe poder reutilizarse en cualquier proyecto agregando un solo archivo (`.claude/settings.json`) y sin copiar los agentes a mano.

## Contexto

- Trabajo con **Claude Code en la web**: cada sesión corre en la nube y clona el repo del proyecto desde GitHub. No hay `~/.claude/` local persistente.
- Flujo de despliegue: push a GitHub → Render despliega en un ambiente de prueba.
- Metodología: *spec-driven development*. Yo escribo `spec.md`, los agentes planean, implementan y revisan.

## Decisión de arquitectura

1. **Mecanismo principal:** plugin + marketplace en este repo. Cada proyecto lo habilita con `extraKnownMarketplaces` + `enabledPlugins` en su `.claude/settings.json`.
2. **Respaldo:** una GitHub Action reutilizable que copia los agentes y comandos a `.claude/` del proyecto y abre un PR. Solo se usa si el plugin no carga en sesiones web.
3. **No usar git submodules.** Las sesiones en la nube pueden no inicializarlos.

> **Importante para Claude Code:** antes de crear `plugin.json`, `marketplace.json` y `settings.json`, verifica el esquema vigente en la documentación oficial de Claude Code (plugins, marketplaces, subagents, slash commands, settings). Si el formato difiere de esta spec, sigue la documentación y anótalo en el README.

## Estructura esperada

```
claude-agentes/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
│   ├── arquitecto.md
│   ├── desarrollador.md
│   └── qa-seguridad.md
├── commands/
│   ├── inicializar-proyecto.md
│   ├── nueva-spec.md
│   ├── planear.md
│   ├── implementar.md
│   └── revisar.md
├── plantillas/
│   ├── spec.md
│   ├── CLAUDE.proyecto.md
│   └── settings.proyecto.json
├── .github/workflows/
│   └── sync-agentes.yml          ← workflow reutilizable (respaldo)
├── ejemplos/
│   └── sync-en-proyecto.yml      ← cómo llamarlo desde un proyecto
├── CHANGELOG.md
└── README.md
```

## Requisitos funcionales

### RF-1. Manifiestos del plugin
- `plugin.json`: nombre `claude-agentes`, versión semántica empezando en `0.1.0`, descripción y autor.
- `marketplace.json`: marketplace llamado `alejandro-agentes` que publica el plugin `claude-agentes` desde la raíz del repo.

### RF-2. Ciclo de vida de una spec
Toda spec vive en `specs/NNN-slug/` dentro del proyecto y tiene un campo `estado` en su frontmatter con estos valores, en orden:

`borrador → planeada → aprobada → en-desarrollo → en-revision → terminada` (o `rechazada`)

- Solo yo (humano) cambio `planeada → aprobada`.
- Los agentes deben respetar el estado y negarse a avanzar si no corresponde.

### RF-3. Agente `arquitecto` (PO/Arquitecto)
- **Modelo:** el más capaz disponible (opus).
- **Herramientas:** Read, Grep, Glob, Write, Edit. Debe escribir **solo** dentro de `specs/`.
- **Responsabilidades:**
  - Leer `spec.md`, `CLAUDE.md` y el código relevante.
  - Detectar ambigüedades y requisitos faltantes y listarlos en una sección `## Preguntas abiertas` de `plan.md`.
  - Generar `plan.md` (enfoque técnico, archivos afectados, riesgos, impacto en datos y migraciones) y `tasks.md` (tareas numeradas y pequeñas, cada una con su criterio de "hecho").
  - Si la spec es demasiado grande (más de unas 8 tareas), proponer dividirla en varias specs.
  - Cambiar el estado a `planeada`.

### RF-4. Agente `desarrollador`
- **Modelo:** sonnet.
- **Herramientas:** todas las de edición y Bash.
- **Responsabilidades:**
  - Trabajar solo con specs en estado `aprobada` o `en-desarrollo`.
  - Crear o usar la rama `spec/NNN-slug`. **Nunca** hacer push a `main`.
  - Implementar `tasks.md` en orden, marcando cada tarea como completada en el archivo.
  - Escribir o actualizar tests por cada tarea y correrlos con el comando definido en el `CLAUDE.md` del proyecto.
  - Un commit por tarea con el mensaje `spec NNN: <tarea>`.
  - Si recibe un `review.md` con estado RECHAZADO, atender solo los hallazgos listados.

### RF-5. Agente `qa-seguridad`
- **Modelo:** sonnet.
- **Herramientas:** Read, Grep, Glob, Bash. **Sin Edit ni Write sobre código.** Solo escribe `specs/NNN-slug/review.md`.
- **Responsabilidades:**
  - Verificar cada criterio de aceptación de `spec.md` (cumple / no cumple / evidencia).
  - Correr la suite completa de tests y el linter si existe.
  - Revisión de seguridad con checklist: inyección (SQL, comandos), validación de entrada, autenticación y autorización, secretos en código o logs, manejo de datos personales y sensibles, dependencias vulnerables (`npm audit` / `dotnet list package --vulnerable` / equivalente según stack), CORS y headers.
  - Clasificar hallazgos como Crítico, Alto, Medio o Bajo.
  - Veredicto: **APROBADO** si no hay Crítico ni Alto y todos los criterios se cumplen; en otro caso **RECHAZADO**.

### RF-6. Comandos
| Comando | Qué hace |
|---|---|
| `/inicializar-proyecto` | Copia al proyecto actual las plantillas: `CLAUDE.md` (si no existe), `.claude/settings.json` y `specs/_plantilla.md`. Pregunta stack, comando de tests y comando de build y llena el `CLAUDE.md`. |
| `/nueva-spec <titulo>` | Crea `specs/NNN-slug/spec.md` desde la plantilla, con número consecutivo y estado `borrador`. |
| `/planear <ruta-spec>` | Invoca al `arquitecto`. Termina mostrando las preguntas abiertas y pidiéndome aprobar. |
| `/implementar <ruta-spec>` | Verifica que el estado sea `aprobada`. Ejecuta `desarrollador` → `qa-seguridad`. Si el veredicto es RECHAZADO, repite el ciclo **máximo 2 veces**; si sigue rechazado, se detiene y me reporta. Si es APROBADO, hace push de la rama y deja el estado en `en-revision` para que yo abra el PR y lo vea en el preview de Render. |
| `/revisar <ruta-spec>` | Ejecuta solo `qa-seguridad` (útil después de cambios manuales). |

### RF-7. Plantillas
- `plantillas/spec.md`: frontmatter (`id`, `titulo`, `estado`, `autor`) y secciones: Objetivo, Contexto/usuarios, Requisitos funcionales, **Criterios de aceptación** (formato verificable: "Dado / Cuando / Entonces"), Fuera de alcance, Notas técnicas, Datos sensibles involucrados (sí/no y cuáles).
- `plantillas/CLAUDE.proyecto.md`: secciones Stack, Comandos (instalar, tests, lint, build, correr local), Convenciones, Despliegue (Render, rama de preview), Reglas de seguridad específicas.
- `plantillas/settings.proyecto.json`: el bloque `extraKnownMarketplaces` + `enabledPlugins` que habilita este plugin desde `tu-usuario/claude-agentes`.

### RF-8. Sincronización de respaldo
- `.github/workflows/sync-agentes.yml` como workflow reutilizable (`workflow_call`) que:
  1. Hace checkout del proyecto y de `claude-agentes` (rama o tag configurable).
  2. Copia `agents/` y `commands/` a `.claude/agents/` y `.claude/commands/` del proyecto.
  3. Si hay cambios, abre un PR con el título `chore: sincronizar agentes vX.Y.Z`.
- `ejemplos/sync-en-proyecto.yml`: cómo llamarlo desde un proyecto con disparo manual (`workflow_dispatch`) y semanal (`schedule`).

### RF-9. README
Debe explicar:
1. Qué es y cuál es el flujo (diagrama de texto del ciclo de una spec).
2. Instalación en un proyecto nuevo (mecanismo principal).
3. Cómo verificar que cargó en una sesión web (`/agents`, `/help`), incluyendo si los comandos aparecen con prefijo (`/claude-agentes:implementar`).
4. Cuándo y cómo usar el respaldo con GitHub Action.
5. Cómo actualizar versiones (CHANGELOG, tags).
6. Cómo sobrescribir un agente en un proyecto específico (mismo nombre en `.claude/agents/` del proyecto).

## Criterios de aceptación

- **CA-1.** Dado un proyecto nuevo con solo `.claude/settings.json` de la plantilla, cuando abro una sesión de Claude Code, entonces `/agents` muestra `arquitecto`, `desarrollador` y `qa-seguridad`.
- **CA-2.** Dado `/nueva-spec login`, entonces se crea `specs/001-login/spec.md` con estado `borrador`.
- **CA-3.** Dada una spec en `borrador`, cuando corro `/planear`, entonces se crean `plan.md` y `tasks.md`, el estado pasa a `planeada` y no se modifica ningún archivo fuera de `specs/`.
- **CA-4.** Dada una spec en `planeada` (no aprobada), cuando corro `/implementar`, entonces se niega y me pide aprobarla.
- **CA-5.** Dada una spec `aprobada`, cuando corro `/implementar`, entonces el trabajo ocurre en la rama `spec/NNN-slug`, hay un commit por tarea, existe `review.md` con veredicto y nunca se hace push a `main`.
- **CA-6.** El agente `qa-seguridad` no tiene herramientas de edición en su frontmatter.
- **CA-7.** Si QA rechaza dos veces seguidas, el flujo se detiene y reporta los hallazgos pendientes.
- **CA-8.** Los JSON del plugin y del marketplace son válidos según la documentación vigente.

## Fuera de alcance

- Integración directa con la API de Render (los previews los maneja Render por PR).
- Agentes adicionales (DevOps, UX, documentador). Se agregan en specs futuras.
- Publicar el plugin en marketplaces de terceros.

## Plan de ejecución sugerido

1. **Fase 1 – Esqueleto:** estructura de carpetas, `plugin.json`, `marketplace.json`, README inicial. Verificar esquemas contra la documentación.
2. **Fase 2 – Agentes:** los tres agentes con sus prompts, herramientas y modelos.
3. **Fase 3 – Comandos y plantillas:** los cinco comandos y las tres plantillas.
4. **Fase 4 – Respaldo:** workflow reutilizable y ejemplo.
5. **Fase 5 – Prueba real:** crear un repo de prueba vacío, aplicar `/inicializar-proyecto`, correr una spec mínima ("endpoint /health que devuelve 200") de punta a punta y documentar el resultado en el README.

## Notas técnicas

- Prompts de los agentes en **español**, claros y con pasos numerados.
- Los agentes deben leer siempre el `CLAUDE.md` del proyecto antes de actuar: ahí está el stack y los comandos de test; los agentes son agnósticos al stack.
- El repo puede ser público: no debe contener secretos.
