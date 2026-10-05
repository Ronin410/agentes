# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/); versiones con [SemVer](https://semver.org/lang/es/).

## [Sin publicar]

### Corregido
- El workflow de respaldo y su ejemplo apuntaban a la rama `main`; este repo usa `master` (`@master`, `ref: master`).

## [0.1.0] - 2026-10-03

### Agregado
- Plugin `agentes` y marketplace `alejandro-agentes` (`.claude-plugin/`).
- Agentes `arquitecto` (opus), `desarrollador` (sonnet) y `qa-seguridad` (sonnet).
- Comandos `/inicializar-proyecto`, `/nueva-spec`, `/planear`, `/implementar` (máx. 2 rondas dev + QA) y `/revisar`.
- Plantillas `spec.md`, `CLAUDE.proyecto.md` y `settings.proyecto.json`.
- Workflow reutilizable `sync-agentes.yml` (respaldo) y ejemplo `ejemplos/sync-en-proyecto.yml`.
- README con flujo, instalación, verificación, respaldo, versionado, sobrescritura de agentes y resultado de la prueba de punta a punta.

### Notas
- El plugin se llama `agentes` en lugar de `claude-agentes` porque los nombres con prefijo `claude-` están reservados (ver README).
