# Cómo usar el plugin `agentes` en un repo tuyo

Guía paso a paso para habilitar el plugin en un proyecto y correr el ciclo completo de una spec.

## Antes de empezar (una sola vez)

1. **Rama por defecto de `ronin410/agentes`.** Ve a GitHub → *Settings → Branches* y deja `master` como rama por defecto. El marketplace se lee de esa rama.
2. **Visibilidad.** Si `agentes` es privado, la sesión web de tu proyecto necesita acceso a él. Lo más simple es dejarlo público, porque no contiene secretos.
3. **(Opcional) Tag de versión.** Crea el tag `v0.1.0`. Así puedes fijar la versión en tus proyectos.

## En tu proyecto

### Paso 1. Habilita el plugin

En tu repo, crea el archivo `.claude/settings.json` con este contenido:

```json
{
  "extraKnownMarketplaces": {
    "alejandro-agentes": {
      "source": { "source": "github", "repo": "ronin410/agentes" }
    }
  },
  "enabledPlugins": {
    "agentes@alejandro-agentes": true
  }
}
```

Haz commit y push **a la rama que usará la sesión web** (normalmente `main`). Si no está en esa rama, la sesión no lo verá.

### Paso 2. Abre una sesión de Claude Code en la web

Abre una sesión sobre ese repo. Si te pide confiar en el marketplace o instalar el plugin, acepta.

### Paso 3. Verifica que cargó

- Escribe `/agents`. Deben aparecer `agentes:arquitecto`, `agentes:desarrollador` y `agentes:qa-seguridad`.
- Escribe `/` o `/help`. Deben aparecer `/agentes:nueva-spec`, `/agentes:planear`, `/agentes:implementar` y los demás.
- Si algo falla, abre `/plugin` y revisa la pestaña **Errors**. Si no carga, ve a la sección "Si no carga el plugin (respaldo)".

### Paso 4. Inicializa el proyecto

```
/agentes:inicializar-proyecto
```

Te preguntará el stack y los comandos de instalar, tests, lint, build y correr en local. Con eso crea `CLAUDE.md` y `specs/_plantilla.md`. **Revisa `CLAUDE.md`**: los agentes dependen de ese archivo. Haz commit y push.

### Paso 5. Crea una spec

```
/agentes:nueva-spec endpoint health
```

Se crea `specs/001-endpoint-health/spec.md` con `estado: borrador`. Ábrela, llénala (objetivo, requisitos y sobre todo los **criterios de aceptación** en formato Dado/Cuando/Entonces) y haz commit y push.

### Paso 6. Planea

```
/agentes:planear specs/001-endpoint-health
```

El arquitecto genera `plan.md` y `tasks.md`, deja la spec en `planeada` y te muestra las **preguntas abiertas**.

### Paso 7. Apruebas tú

1. Lee `plan.md` y `tasks.md`.
2. Responde las preguntas abiertas, agregando la respuesta a la spec o al plan.
3. En `spec.md` cambia `estado: planeada` por `estado: aprobada`.
4. Haz commit y push.

Este paso es solo tuyo: si la spec sigue en `planeada`, `/implementar` se niega.

### Paso 8. Implementa

```
/agentes:implementar specs/001-endpoint-health
```

Esto hace lo siguiente:

- El desarrollador trabaja en la rama `spec/001-endpoint-health`, con un commit y tests por tarea.
- QA revisa y escribe `review.md`.
- Si QA rechaza, hay una segunda ronda. Si rechaza otra vez, el flujo se detiene y te reporta los hallazgos.
- Si QA aprueba, se hace push de la rama y la spec queda en `en-revision`.

### Paso 9. Cierra el ciclo

1. Abre el PR de `spec/001-endpoint-health` hacia `main`.
2. Revisa el preview que Render genera para el PR.
3. Haz merge y cambia el estado de la spec a `terminada`.

### Revisión manual (opcional)

Si cambias código a mano, corre `/agentes:revisar specs/001-endpoint-health` para otra pasada de QA.

## Si no carga el plugin (respaldo)

1. Copia `ejemplos/sync-en-proyecto.yml` de `agentes` a `.github/workflows/sync-agentes.yml` en tu proyecto.
2. En tu proyecto, ve a *Settings → Actions → General* y activa **Allow GitHub Actions to create and approve pull requests**.
3. Ejecuta el workflow desde la pestaña Actions y mergea el PR que abre.
4. Desde ahí los comandos se llaman sin prefijo (`/implementar`, `/planear`).

> **Pendiente:** el workflow y el ejemplo apuntan a la rama `main`, pero `agentes` usa `master`. Hay que cambiar `@main` y `ref: main` a `master` antes de usar el respaldo.

## Cosas a tener en cuenta

- **Las sesiones web son efímeras.** Todo lo que no esté commiteado y pusheado se pierde, así que haz push después de cada paso (specs, aprobación, `CLAUDE.md`).
- **Pendiente de verificar.** La carga del plugin en una sesión web real no se ha probado todavía. El flujo completo se probó en local, con éxito. Si en el paso 3 algo no aparece, revisa `/plugin` → **Errors**.
- **Solo tú apruebas.** Ningún agente cambia una spec de `planeada` a `aprobada`.
- **Nunca se hace push a `main`.** Todo el trabajo va en `spec/NNN-slug` y entra por PR.
