# arness-back (`@linktic/arness-back`)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/arness-back`
> Tipo: **herramienta interna (harness de agentes IA / CLI)** · Cliente: **interno Linktic BI Analytics** (transversal a los backends)

## Descripción / propósito

Paquete que distribuye un **harness de agentes IA** (Claude Code, Codex, Gemini CLI, Cursor) para los **backends NestJS de analítica**. Incluye reglas únicas (`harness/AGENTS.md`), script de verificación de estado (`harness/init.sh`), agentes `lider`, `implementador` y `revisor`, la skill `nest-base` (crea la base de un proyecto NestJS con la convención del área), skill `commit-hygiene`, playbooks (nuevo módulo, nuevo endpoint, agregar base de datos, revisión) y referencias (configuración, contrato HTTP, plantilla de progreso).

Se instala **desde git** (sin registro npm), fijando un tag `vX.Y.Z`; un CLI `arness` copia la base al repo destino y crea los "puentes" para cada herramienta.

## Stack y versiones

- Node.js >= 20 (CI usa Node 24), ESM (`"type": "module"`), **sin dependencias** (solo módulos nativos de Node).
- pnpm para instalar en los proyectos destino.
- Versión actual: `0.1.0` (tag `v0.1.0`, CHANGELOG 2026-09-25).

## Arquitectura y estructura de carpetas

```
bin/arness.mjs          # CLI: init, sync, status, verify, decision new
harness/
├── AGENTS.md, init.sh  # reglas y verificación (gestionados por el paquete)
├── agents/             # lider.md, implementador.md, revisor.md
├── skills/             # nest-base (con templates eslint/prettier/editorconfig), commit-hygiene
├── playbooks/          # add-database, new-endpoint, new-module, review
├── reference/          # configuracion, contrato-http, progress-plantilla
└── decisiones/         # README + _plantilla (las decisiones numeradas pertenecen al repo instalador)
docs/adr/               # ADRs del propio harness (0001 registro, 0002 carpeta propia, 0003 paquete, 0004 instalación desde git)
test/cli.test.mjs       # tests del CLI
CLAUDE.md, GEMINI.md, .cursor/rules/agents.mdc   # puentes hacia harness/AGENTS.md
```

## Funcionalidades (CLI)

| Comando | Qué hace |
|---|---|
| `arness init` | Copia la base a `harness/`, crea `harness/decisiones/`, puentes (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursor/rules/`, `.claude/{agents,skills}`, `.agents/skills`) vía symlink o copia, escribe manifiesto `harness/.arness.json` (versión + hash por archivo) y agrega `harness/progress.md` al `.gitignore` |
| `arness sync [--dry-run] [--force] [--copy]` | Actualiza archivos del manifiesto; se detiene si alguien editó la base; nunca toca decisiones ni progress |
| `arness status` | Versión instalada e integridad |
| `arness verify` | Ejecuta `harness/init.sh` |
| `arness decision new "<título>"` | Crea una decisión numerada en `harness/decisiones/` |

Instalación en un backend existente:
```bash
pnpm add -D "git+https://github.com/bi-analytics-linktic/harness-back.git#v0.1.0"
pnpm arness init && ./harness/init.sh
```
En repo nuevo sin `package.json`: clonar el tag a un directorio temporal y ejecutar `node bin/arness.mjs init`, luego pedir al agente la skill `nest-base`.

## Fuentes de datos e integraciones

No accede a datos. Integra con herramientas de agentes (Claude Code, Codex, Gemini CLI, Cursor) mediante archivos puente. Requiere acceso de lectura al repo GitHub `bi-analytics-linktic/harness-back`.

## Cómo ejecutarlo localmente

```bash
pnpm test            # node --test (tests del CLI)
pnpm verify          # ./harness/init.sh (en este repo verifica harness y CLI)
pnpm pack:check      # npm pack --dry-run: archivos que se distribuyen
```
Publicar versión: actualizar `CHANGELOG.md`, `pnpm version <patch|minor|major>`, `git push --follow-tags`.

## Variables de entorno

Ninguna.

## Pruebas

`test/cli.test.mjs` con el runner nativo `node --test`. CI (`azure-pipelines.yml`, trigger en `main` y tags `v*`, PR a `main`): Node 24, `corepack enable`, `pnpm test`, `./harness/init.sh --quick`, y validación de que el tag coincida con `package.json`.

## Estado git

- Remoto: `https://github.com/bi-analytics-linktic/harness-back.git` (el directorio local se llama `arness-back`, el repo `harness-back`)
- Rama actual: `feat/package-git-install` (también `main`; ambas en origin)
- Tags: `v0.1.0`
- Commits: `0a61fe0` (2026-09-25) "feat(package): distribute harness as @linktic/arness-back installed from git", `ba4d585` "refactor(harness): move agent harness into dedicated harness/ folder", `c3b30ac` "implementacion harness base"
- Árbol de trabajo limpio.

## Notas

- Pensado para estandarizar los backends NestJS de Analítica (p. ej. `rdt-ms-reporter-bi`, backs de Balu, AgoraCRM y Docum), que comparten el patrón NestJS + `pg` solo lectura + Redis.
- `rdt-ms-reporter-bi` tiene su propia skill de scaffolding en `.claude/skills/` (antecesora de este enfoque); los MVP de `sistema-ciudadano` también contienen una carpeta `harness/` propia.
- El README recomienda hacer commit de `harness/` en el repo destino para que funcione recién clonado, antes de `pnpm install`.
