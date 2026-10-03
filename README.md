# agentes — plugin de agentes por roles para Claude Code

Plugin de [Claude Code](https://code.claude.com/docs) con un equipo estandarizado de agentes para **spec-driven development**:

| Agente | Modelo | Herramientas | Rol |
|---|---|---|---|
| `arquitecto` | opus | Read, Grep, Glob, Write, Edit (solo escribe en `specs/`) | Convierte `spec.md` en `plan.md` + `tasks.md` |
| `desarrollador` | sonnet | edición + Bash | Implementa `tasks.md` en la rama `spec/NNN-slug`, un commit y tests por tarea |
| `qa-seguridad` | sonnet | Read, Grep, Glob, Bash (sin Edit/Write) | Verifica criterios, corre tests y escribe `review.md` con veredicto |

Incluye 5 comandos (`/inicializar-proyecto`, `/nueva-spec`, `/planear`, `/implementar`, `/revisar`), plantillas y una GitHub Action de respaldo.

> **Diferencias con la spec 000** (verificadas contra la documentación oficial y `claude plugin validate` v2.1.289):
>
> 1. **El plugin se llama `agentes`, no `claude-agentes`.** La documentación reserva los nombres que empiezan con `claude-`, y `claude plugin validate` los rechaza (*"Plugin name ... is reserved: it passes as one of Anthropic's own"*). Con `agentes` coincide con el nombre del repo (`ronin410/agentes`). El marketplace sí conserva el nombre `alejandro-agentes`.
> 2. **El repositorio es `ronin410/agentes`** (la plantilla usa ese valor en lugar de `tu-usuario/claude-agentes`).
> 3. **Comandos con prefijo.** Los comandos y agentes de un plugin aparecen con el nombre del plugin como prefijo: `/agentes:implementar`, `agentes:arquitecto`. Si no hay otro comando con el mismo nombre, también funciona sin prefijo (`/implementar`).
> 4. **`commands/` vs `skills/`.** La documentación recomienda `skills/` para plugins nuevos, pero `commands/` sigue soportado y funciona igual; se mantuvo `commands/` como pide la spec.
> 5. **"Solo escribe en `specs/`"** (arquitecto) se garantiza por instrucciones del prompt, no por permisos: los agentes de plugin ignoran `hooks` y `permissionMode`, así que no hay forma de restringir rutas desde el plugin. En la prueba real (abajo) el arquitecto no tocó nada fuera de `specs/`.
> 6. **`qa-seguridad` escribe `review.md` con Bash** (heredoc), porque la spec le prohíbe tener Edit/Write en su frontmatter (CA-6).

---

## 1. Qué es y cuál es el flujo

Tú escribes la spec; los agentes planean, implementan y revisan. Toda spec vive en `specs/NNN-slug/` y tiene un `estado` en su frontmatter:

```
  Tú                    arquitecto               Tú                  desarrollador + qa-seguridad              Tú
  ──                    ──────────               ──                  ────────────────────────────              ──
/nueva-spec  ─►  borrador ──/planear──► planeada ──(apruebas)──► aprobada ──/implementar──► en-desarrollo
                                          │                                                     │
                                  plan.md, tasks.md,                         ┌──────────────────┘
                                  preguntas abiertas                         ▼
                                                                desarrollador (rama spec/NNN-slug,
                                                                1 commit + tests por tarea)
                                                                             │
                                                                             ▼
                                                                qa-seguridad ─► review.md
                                                                     │                │
                                                              APROBADO          RECHAZADO
                                                                     │                │
                                                                     │       ronda 2: desarrollador
                                                                     │       atiende solo hallazgos
                                                                     │       → qa-seguridad
                                                                     │                │
                                                                     │       ¿RECHAZADO otra vez?
                                                                     │       → se detiene y te reporta
                                                                     ▼
                                                    push de spec/NNN-slug, estado en-revision
                                                                     │
                                                     Tú abres el PR → preview en Render → merge
                                                                     ▼
                                                                 terminada   (o rechazada)
```

Reglas clave:

- Solo tú cambias `planeada → aprobada`.
- Nadie hace push a `main`; todo va en `spec/NNN-slug` y entra por PR.
- Los agentes leen siempre el `CLAUDE.md` del proyecto (stack, comando de tests, lint, build). Son agnósticos al stack.

## 2. Instalación en un proyecto (mecanismo principal)

Agrega este archivo al proyecto como `.claude/settings.json` (es `plantillas/settings.proyecto.json`), haz commit y push:

```json
{
  "extraKnownMarketplaces": {
    "alejandro-agentes": {
      "source": {
        "source": "github",
        "repo": "ronin410/agentes"
      }
    }
  },
  "enabledPlugins": {
    "agentes@alejandro-agentes": true
  }
}
```

Para fijar una versión agrega `"ref": "v0.1.0"` dentro de `source`.

Después, en una sesión de Claude Code sobre el proyecto:

1. `/agentes:inicializar-proyecto` → te pregunta stack y comandos, crea `CLAUDE.md`, `specs/_plantilla.md` y completa `.claude/settings.json`.
2. `/agentes:nueva-spec login` → crea `specs/001-login/spec.md` en `borrador`. Llénala.
3. `/agentes:planear specs/001-login` → `plan.md`, `tasks.md` y preguntas abiertas.
4. Responde las preguntas, cambia `estado: aprobada` y haz commit.
5. `/agentes:implementar specs/001-login` → rama, commits, `review.md` y push si QA aprueba.
6. Abre el PR, revisa el preview de Render y mergea. Marca `estado: terminada`.
7. `/agentes:revisar specs/001-login` cuando hagas cambios manuales y quieras otra revisión de QA.

> Los comandos copian archivos desde la carpeta del plugin y crean carpetas: según tu modo de permisos, Claude te pedirá aprobación la primera vez.

## 3. Verificar que cargó en una sesión web

1. Abre una sesión de Claude Code en la web sobre el proyecto (el repo debe tener el `.claude/settings.json` ya en la rama que se clona).
2. Si es la primera vez, Claude Code puede pedirte confiar en el marketplace / instalar el plugin: acepta.
3. Corre `/agents`: deben aparecer `agentes:arquitecto`, `agentes:desarrollador` y `agentes:qa-seguridad`.
4. Corre `/help` o escribe `/` : deben aparecer `/agentes:inicializar-proyecto`, `/agentes:nueva-spec`, `/agentes:planear`, `/agentes:implementar`, `/agentes:revisar`. Sin conflicto de nombres también funcionan sin prefijo (`/implementar`).
5. `/plugin` muestra el estado del plugin y, en la pestaña **Errors**, cualquier problema de carga.

Si nada de esto aparece, usa el respaldo (sección 4).

## 4. Respaldo con GitHub Action

Úsalo **solo si el plugin no carga** en las sesiones web. Copia `agents/`, `commands/` y `plantillas/` dentro del `.claude/` del proyecto (como agentes y comandos de proyecto, sin prefijo) y abre un PR `chore: sincronizar agentes vX.Y.Z`.

1. Copia `ejemplos/sync-en-proyecto.yml` a `.github/workflows/sync-agentes.yml` en tu proyecto.
2. En el proyecto: **Settings → Actions → General → Workflow permissions** → activa *Allow GitHub Actions to create and approve pull requests*.
3. Si `ronin410/agentes` es privado, en **este** repo: **Settings → Actions → General → Access** → permite el acceso desde repos de tu cuenta. (Público no requiere nada.)
4. Ejecútalo manualmente (**Actions → Sincronizar agentes → Run workflow**, puedes indicar rama o tag) o espera la corrida semanal (lunes).
5. Revisa y mergea el PR.

En modo respaldo los comandos se llaman sin prefijo (`/implementar`) y las plantillas se leen de `.claude/plantillas/`. Si usas el respaldo, quita el bloque del plugin de `.claude/settings.json` para no tener agentes duplicados (los de `.claude/agents/` tendrían prioridad de todos modos).

No se usan git submodules porque las sesiones en la nube pueden no inicializarlos.

## 5. Actualizar versiones

1. Haz los cambios en este repo.
2. Sube `version` en `.claude-plugin/plugin.json` siguiendo semver (`0.1.0 → 0.1.1` correcciones, `0.2.0` funcionalidad nueva, `1.0.0` cambios incompatibles). Mientras `version` no cambie, quienes ya lo instalaron se quedan en la versión anterior.
3. Anota el cambio en `CHANGELOG.md`.
4. Valida: `claude plugin validate .`
5. Commit, tag y push:

   ```bash
   git tag v0.2.0 && git push origin main --tags
   ```

6. Los proyectos sin `ref` toman la nueva versión al iniciar una sesión nueva; los que fijaron `"ref": "v0.1.0"` deben actualizar el `ref`. En respaldo, corre el workflow con el nuevo tag.

## 6. Sobrescribir un agente en un proyecto

Los agentes de `.claude/agents/` del proyecto tienen prioridad sobre los del plugin cuando comparten nombre:

1. Copia el agente: `agents/desarrollador.md` → `<proyecto>/.claude/agents/desarrollador.md`.
2. Ajusta el prompt, las herramientas o el modelo. Mantén `name: desarrollador`.
3. Los comandos delegan primero en el agente con nombre simple (`desarrollador`), así que usarán tu versión. El del plugin sigue disponible como `agentes:desarrollador`.

Lo mismo aplica a comandos: un `.claude/commands/implementar.md` en el proyecto se invoca como `/implementar`, y el del plugin sigue como `/agentes:implementar`.

## Prueba real (fase 5)

Ejecutada el 2026-10-03 con Claude Code 2.1.289 en un repo vacío de Node 22 (sin dependencias), con el plugin instalado desde este marketplace y un remoto git local:

| Paso | Resultado |
|---|---|
| `claude plugin validate .` (marketplace y plugin) | ✔ Validation passed |
| `claude plugin details agentes` | 3 agentes y 5 comandos cargados |
| `/agentes:nueva-spec login` | `specs/001-login/spec.md`, `estado: borrador` (CA-2 ✔) |
| `/agentes:planear specs/001-health` (spec "endpoint /health que devuelve 200") | `plan.md` + `tasks.md` (3 tareas), 5 preguntas abiertas, `estado: planeada`, `git status` solo con cambios en `specs/` (CA-3 ✔) |
| `/agentes:implementar` con la spec en `planeada` | Se negó y pidió aprobarla; sin ramas ni commits (CA-4 ✔) |
| `/agentes:implementar` con la spec `aprobada` | Rama `spec/001-health`, un commit por tarea (`spec 001: ...`). **Ronda 1: QA RECHAZADO** (hallazgo Alto real: `GET //` tumbaba el proceso). **Ronda 2:** el desarrollador corrigió solo ese hallazgo, QA **APROBADO**. Push de `spec/001-health`, `estado: en-revision`, `main` del remoto intacto (CA-5 ✔) |
| `npm test` en la rama | 4/4 pasan |

Ajuste derivado de la prueba: QA reportaba como hallazgo "Bajo" el cambio de estado `aprobada → en-desarrollo`, que es parte del flujo; se aclaró en su prompt.

No verificado en vivo: CA-1 en una sesión **web** (requiere que este repo esté en GitHub en la rama por defecto) y CA-7 (dos rechazos seguidos: la lógica está en `/implementar`, pero en la prueba QA aprobó en la ronda 2).

## Estructura

```
.claude-plugin/plugin.json       manifiesto del plugin (agentes 0.1.0)
.claude-plugin/marketplace.json  marketplace alejandro-agentes
agents/                          arquitecto, desarrollador, qa-seguridad
commands/                        inicializar-proyecto, nueva-spec, planear, implementar, revisar
plantillas/                      spec.md, CLAUDE.proyecto.md, settings.proyecto.json
.github/workflows/sync-agentes.yml  workflow reutilizable (respaldo)
ejemplos/sync-en-proyecto.yml    cómo llamarlo desde un proyecto
```

Este repo es público-seguro: no contiene secretos.
