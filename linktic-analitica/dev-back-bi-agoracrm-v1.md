# dev-back-bi-agoracrm-v1

**Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/dev-back-bi-agoracrm-v1`
**Tipo:** Backend · **Cliente/producto:** AgoraCRM · **Área:** BI Analytics (Linktic)

## Descripción / propósito

Es el backend de analítica (BI) de AgoraCRM. Expone reportes **de solo lectura** sobre la base PostgreSQL del CRM, que es un Odoo (tablas `crm_lead`, `sale_order`, `res_partner`, etc.), para que los use un frontend de BI. No es transaccional: no crea, modifica ni borra datos de negocio. Está montado sobre el harness interno `@linktic/arness-back` (ver `arness-back`).

## Stack y versiones

| Pieza | Versión |
|---|---|
| Node.js | >= 24 (`.nvmrc`: v24.21.0) |
| NestJS (common/core/config/jwt/platform-express/terminus) | ^12 |
| TypeScript | ^6.0.2, ESM (`"type": "module"`) |
| PostgreSQL (driver `pg`) | ^8.23 |
| Redis (`ioredis`), caché opcional | ^6 |
| Logging | `nestjs-pino` ^5.2 / `pino-http` ^11 |
| Validación | `class-validator` ^0.15 / `class-transformer` ^0.5 |
| Tests | Vitest ^4.1 + Supertest ^7 (+ `@vitest/coverage-v8`) |
| Lint/format | ESLint ^10, Prettier ^3 |
| Harness | `@linktic/arness-back` v0.1.0 (git: `bi-analytics-linktic/harness-back`) |
| Gestor de paquetes | pnpm 12.6.0 |

## Arquitectura y estructura de carpetas

La arquitectura es hexagonal por feature, con la organización **cliente → módulo → endpoint**: controller → service → repository → sql (más `columns` y `entity`).

```
src/
  main.ts, app.module.ts, app.setup.ts   # prefijo global api/v1 (health queda fuera)
  config/          # ConfigModule + validación de env
  common/
    auth/          # login/refresh JWT, guard global, decorador @Public
    cache/         # Redis opcional (cache-key, CacheService)
    database/      # pools PostgreSQL por conexión (DATABASES), helpers SQL
    health/        # /health (terminus)
    http/          # respuesta { data, meta }, ApiException, filtro global, validación
    logging/       # pino, request-id, redacción de datos sensibles
  modules/agoracrm/
    crm-leads/     # crm-lead/ y branch/ (sucursales)
    sale-orders/   # sale-order/
test/              # e2e (auth, health, crm-lead, branch, sale-order)
harness/           # reglas para agentes, playbooks, decisiones (ADR), init.sh
knowledge/         # contexto generado con kaddo (negocio, producto, tech, roadmap)
postman/           # colección y environment local
```

Documentación del repo: `README.md`, `README_BACK_AGORA_1_2_3.md` (contexto, parte 1), `FRONT.md` (guía para el frontend) y `harness/decisiones/` (ADR 0001 sobre NestJS 12 con ESM, Vitest y ESLint; ADR 0002 sobre el total estimado en la paginación, todavía en estado propuesta). `readme_context_2.md` y `readme_context_3.md` existen pero están vacíos. `CLAUDE.md`, `GEMINI.md` y `.cursor/rules` solo apuntan a `harness/AGENTS.md`.

## Endpoints

| Método | Ruta | Descripción | Origen | Caché |
|---|---|---|---|---|
| GET | `/health` | Health check (público) | Postgres/Redis | - |
| POST | `/api/v1/auth/login` | Login con la contraseña compartida; devuelve `accessToken` y `refreshToken` (público) | `ACCESS_TOKEN` | - |
| POST | `/api/v1/auth/refresh` | Renueva el par de tokens (público) | JWT | - |
| GET | `/api/v1/crm-leads` | Leads/oportunidades del CRM (dataset completo, sin paginar) | `public.crm_lead` + JOINs a `product_category`, `utm_campaign`, `crm_branch`, `res_users`, `res_partner` | 300 s |
| GET | `/api/v1/crm-leads/sucursales` | Catálogo de sucursales que tienen leads | `DISTINCT crm_lead` ⨝ `crm_branch` | 300 s |
| GET | `/api/v1/sale-orders` | Cotizaciones y órdenes de venta de Odoo, con su `opportunity_id` → `crm_lead.id` | `public.sale_order` | 300 s |

Todos los endpoints exigen `Authorization: Bearer <accessToken>`, salvo login, refresh y health. Las respuestas tienen la forma `{ data, meta? }` y los errores traen `error.code` (por ejemplo `TOKEN_EXPIRED` o `INVALID_CREDENTIALS`).

## Fuentes de datos e integraciones

- **PostgreSQL** de AgoraCRM (esquema Odoo), conexión lógica `MAIN`, solo lectura. Admite varias conexiones declaradas en `DATABASES` con variables `DB_<NOMBRE>_*`.
- **Redis** como caché opcional: si está apagado o caído, la API responde directo desde PostgreSQL.
- Sin BigQuery ni DuckDB.

## Cómo ejecutarlo localmente

```bash
nvm use && corepack enable
pnpm install
# credenciales locales en .env.local
pnpm start:dev          # nest start --watch (puerto PORT, por defecto 3000)
pnpm build && pnpm start:prod
pnpm lint | lint:check | format | format:check
pnpm verify             # ./harness/init.sh (antes y después de trabajar)
pnpm harness:status | harness:sync   # CLI arness
```

- **Docker:** `Dockerfile` multi-stage sobre `node:24-alpine`. Expone el puerto 3000, corre como usuario `node` y trae HEALTHCHECK sobre `/health`.
- **CI:** `azure-pipelines.yml` (trigger y PR sobre `main`). Corre install, lint:check, format:check, build, tests unitarios, e2e y el build de la imagen Docker.

## Variables de entorno (solo nombres)

Definidas en `.env.example`. `.env` y `.env.local` también existen localmente y están en `.gitignore`.

- App: `PORT`, `LOG_LEVEL`
- BD: `DATABASES`, `DB_MAIN_HOST`, `DB_MAIN_PORT`, `DB_MAIN_NAME`, `DB_MAIN_USER`, `DB_MAIN_PASSWORD`, `DB_MAIN_SSL`, `DB_MAIN_POOL_MAX`, `DB_MAIN_STATEMENT_TIMEOUT_MS`
- Auth: `ACCESS_TOKEN`, `JWT_ACCESS_SECRET`, `JWT_ACCESS_EXPIRES_IN`, `JWT_REFRESH_SECRET`, `JWT_REFRESH_EXPIRES_IN`
- Caché: `CACHE_ENABLED`, `REDIS_URL`, `CACHE_DEFAULT_TTL_SECONDS`, `CACHE_KEY_PREFIX`, `CACHE_OP_TIMEOUT_MS`

## Pruebas

- Unitarias: specs junto al código (config, database, cache, auth, etc.) con `pnpm test`, `test:watch` y `test:cov`.
- E2E: `test/*.e2e-spec.ts` (auth, health, crm-lead, branch, sale-order) con `pnpm test:e2e` (`vitest.config.e2e.ts`).
- Hay unos 44 archivos de spec en total. Para pruebas manuales está la colección Postman en `postman/`.

## Estado git

- **Remote:** `origin` → Azure DevOps `Linktic/BI Analytics/_git/dev-back-bi-agoracrm-v1`
- **Rama actual:** `feature/codebase`, sincronizada con `origin/feature/codebase` y 3 commits por delante de `main` (todavía sin merge a `main`)
- **Últimos commits:**
  - `f2d894a` 2026-10-15 John Hernandez: se agregaron dos endpoints, sucursales y sale orders
  - `b3c4ea8` 2026-09-30 John Hernandez: merge de origin/main en feature/codebase
  - `dafbd43` 2026-09-30 John Hernandez: first commit
  - `7caa623` 2026-09-29 Brayan Suarez Salazar: Added README.md
- Árbol de trabajo limpio.

## Notas

- **Par front↔back:** `dev-back-bi-agoracrm-v1` (back) ↔ `dev-front-bi-agoracrm-v1` (front Next.js 16, por ahora solo scaffold).
- Es un consumidor del harness `arness-back` (`@linktic/arness-back`): los archivos de `harness/` están versionados por hash en `harness/.arness.json` y no se editan a mano.
- El ADR 0002 menciona `public.log_bancario` y `/api/v1/logs/bancario`, que no existen en este repo. Parece venir de otro proyecto que usa el mismo harness (posiblemente Balu).
- La postman se llama `arness-back.postman_collection.json` porque viene de la plantilla del harness.
