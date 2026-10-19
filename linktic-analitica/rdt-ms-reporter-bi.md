# rdt-ms-reporter-bi

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/rdt-ms-reporter-bi`
> Tipo: **back** · Cliente: **Fiduprevisora** (proyecto Azure DevOps "384 - CORE FIDUPREVISORA")

## Descripción / propósito

API REST **de solo lectura** que expone los reportes BI del ERP de Fiduprevisora (generador de reportes). Cada endpoint lee una tabla o, mayoritariamente, una **vista materializada** de PostgreSQL y devuelve las filas tal cual (claves `snake_case`) dentro de un envelope común con `count`, `module_name`, `report_name` y `query_id`. Hoy expone unos **79 reportes** repartidos en módulos de negocio (terceros, negocios fiduciarios, tesorería, embargos, conciliación bancaria, portal de pagos, comisiones, FIC, tributario, presupuesto, cuentas por pagar, facturación y cartera, usuarios/seguridad, contratación derivada, activos fijos).

Nombre en `package.json`: `bi-back-fiduprevisora-board-v1`.

## Stack y versiones

- Node.js >= 22, pnpm 11 (`packageManager: pnpm@11.20.0`)
- NestJS 11 (`@nestjs/common`, `core`, `config` 4, `swagger` 11, `throttler` 6, `platform-express` / Express 5)
- PostgreSQL vía `pg` 8 (**sin ORM**)
- Redis vía `ioredis` 5 (caché Cache-Aside, encabezado `X-Cache`)
- `helmet`, `class-validator`, `class-transformer`
- TypeScript 5.7, Jest 30 + ts-jest, supertest, ESLint 9, Prettier 3

## Arquitectura y estructura de carpetas

```
src/
├── main.ts, app.module.ts, app.setup.ts   # bootstrap, ValidationPipe global, query parser 'extended'
├── common/      # config (EnvConfigService, db-schema), filtros de error, contexto, códigos de error
├── database/    # pool.factory, pool-registry (multi-conexión), query-gate (control de admisión), sql-safety (assertReadOnlySql)
├── cache/       # cliente Redis, servicio de caché, generación de claves
├── logs/        # middleware que registra verbo+URL en request.txt
└── modules/
    ├── health/          # GET /health
    ├── cache-admin/     # POST /api/v1/cache/clear-cache
    └── fiduprevisora/   # un submódulo por área de negocio (controller/service/repository/sql/tables/dto)
test/            # e2e (no requieren BD)
docs/es/         # documentación técnica (intro, config, arquitectura, seguridad, BD, caché, logs, rendimiento)
FRONT.md         # contrato de API para el front (fuente de verdad)
mejoras/         # propuestas NO implementadas, por módulo + diagnóstico (incluye catálogo .xlsx)
postman/         # colección y entornos (local, qa)
scripts/list-views.js  # inventario de vistas/materializadas (CLI, solo lectura)
.claude/skills/  # skill para scaffolding de nuevos módulos (create-new-module-base-structure)
```

Reglas críticas (de `CLAUDE.md`): solo `SELECT`/`WITH` (validado por `assertReadOnlySql` y sesiones `default_transaction_read_only=on`), parámetros posicionales siempre, identificadores solo desde registros congelados en código, `process.env` solo dentro de `EnvConfigService`, esquema configurable (default `bi`) con excepciones por conexión (`tax_service_v2`, `public`).

## Funcionalidades / endpoints

Prefijo global `/api/v1` (salvo `/health`). Sin autenticación (el control de acceso vive en red/proxy/gateway). Casi ningún query param: solo `page`/`pageSize` en `thirdparties` y `fiduciary-transactions/master-list|guarantees`; el resto tiene DTO vacío que rechaza con 400 cualquier parámetro (el filtrado se hace en el cliente).

| Módulo (submódulo) | Ruta base | Reportes |
|---|---|---|
| Salud | `GET /health` | estado del servicio, BD y Redis |
| Caché | `POST /api/v1/cache/clear-cache` | vacía claves bajo el prefijo |
| Terceros (`thirdparties`) | `/thirdparties` | listado maestro (Q1) |
| Negocios fiduciarios + Tesorería (`fiduciary-transactions`) | `/fiduciary-transactions/*` | master-list, guarantees, participants, policies, transfers, real-estate stages, property sales/withdrawals, users per trust, status history, treasury movements, bank-accounts, balances, income-records, checks, cash-flow-summary, movement-origins |
| Embargos (`seizure`) | `/seizure/seizures` | 1 |
| Conciliación bancaria | `/bank-reconciliation/reconciliation-progress` | 1 |
| Portal de pagos | `/portal-payment/portal-payment-requests` | 1 |
| Liquidación de comisiones | `/commission-settlement/*` | 8 |
| FIC | `/fic/*` | 4 |
| Tributario | `/tax-related/*` | 5 |
| Presupuesto | `/budget/*` | 7 |
| Cuentas por pagar | `/accounts-payable/*` | 8 |
| Facturación y cartera | `/billing-collections/*` | 9 |
| Usuarios, roles y seguridad | `/users/*` | 7 |
| Contratación derivada | `/derivative-contracting/*` | 5 |
| Activos fijos | `/fixed-assets/*` | 5 |

Envelope de error uniforme: `{ "error": { "code", "message" } }`. Swagger en `http://localhost:3001/docs` solo con `NODE_ENV=local`.

## Fuentes de datos e integraciones

- **PostgreSQL multi-base**: una conexión principal (`DB_*`) y conexiones adicionales declaradas con una línea `DB_<MODULO>_NAME=<base>` (seizure, fic, budget, tax_related, accounts_payable, billing_collections, users, logs, fixed_assets, derivative_contracting, etc.). La mayoría de reportes leen vistas materializadas (`mv_*`, `view_rep_nfi_*`).
- **Redis** opcional para caché (TTL configurable).
- **AWS**: pipeline construye imagen y la sube a ECR (`data/reporter-backend`, conexión de servicio `AwsFiduDev`, `us-east-1`).

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env      # completar credenciales DB_* y opcionalmente Redis
pnpm run dev              # watch, puerto 3001 (alias de start:dev)
pnpm run build && pnpm run start:prod
node scripts/list-views.js [filtro] [--db=<base>]   # diagnóstico de vistas
```

Docker: `docker build -t bi-back-fiduprevisora-board .` y `docker run --env-file .env -p 3001:3001 ...` (multi-stage node:22-alpine, usuario sin privilegios, HEALTHCHECK a `/health`).

## Variables de entorno (solo nombres)

`NODE_ENV`, `PORT`, `CORS_ORIGINS`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSL`, `SHEMA` (alias `SCHEMA`/`DB_SCHEMA`), `DB_<MODULO>_NAME` (ACCOUNTABILITY, ACCOUNTS_PAYABLE, BANK_RECONCILIATION, BILLING_COLLECTIONS, BUDGET, COMMISSION_SETTLEMENT, DERIVATIVE_CONTRACTING, FIC, FIDUCIARY_TRANSACTIONS, FINANCIAL_OBLIGATIONS, FIXED_ASSETS, LOGS, NORMATIVE, PORTAL_PAYMENT, PORTFOLIO, SARLAFT, SCHEDULES, SEIZURE, TAX_RELATED, USERS), esquemas por conexión `DB_TAX_RELATED_SCHEMA`, `DB_BUDGET_SCHEMA`, `DB_ACCOUNTS_PAYABLE_SCHEMA`, `DB_BILLING_COLLECTIONS_SCHEMA`, `DB_FIDUCIARY_TRANSACTIONS_SCHEMA`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_DB`, `CACHE_ENABLED`, `CACHE_KEY_PREFIX`, `CACHE_TTL_SECONDS`, `REQUEST_LOG_ENABLED`, `REQUEST_LOG_FILE`, `REQUEST_LOG_MAX_BYTES`.

> Existe un `.env` real en el directorio local (no versionar ni copiar valores).

## Pruebas

- `pnpm run test` (unitarias Jest, ~545 archivos `*.spec.ts` entre `src/` y `test/`), `pnpm run test:cov`, `pnpm run test:e2e` (no requieren BD).
- Specs clave: `sql-safety.spec.ts`, `query-gate.spec.ts`, `*-query.dto.spec.ts` (contrato de DTO vacíos), specs de SQL/tablas por módulo.
- Colección Postman en `postman/` con entornos `local.fidu.json` y `qa.fidu.json`.

## Estado git

- Remoto: `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_report_generator`
- Rama actual: `feature/reports-materialize-view` (ramas: `main`, `dev`, `feature/back-fidu`, `feature/reports`)
- Último commit: `4f9c09f` (2026-09-24) "se implementaron nuevos reportes de nuevos modulos, usuarios, logs y trcontract-additions"
- Commits recientes: billing-collections, refactor multiconexión, alineación de nombres de reportes con el Excel, Presupuesto, Cuentas por pagar, Tributario.
- Cambios sin commitear: 3 archivos en `mejoras/diagnostico/` (incluido el catálogo .xlsx).
- Pipeline (`azure-pipelines.yml`): trigger en `dev`, build Docker y push a ECR.

## Notas

- **Relación con `repot-temp`**: ambos repos comparten el mismo remoto (`rdt_ms_api_erp_report_generator`). `repot-temp` es un clon checkout del **commit raíz** `51cb183` ("Commit inicial", 2026-08-27), que es también el primer commit de la historia de este repo. Este repo (`rdt-ms-reporter-bi`) es el **de trabajo activo** (36 commits en su rama, 15 submódulos de negocio); `repot-temp` es una instantánea temporal de la plantilla inicial. Ver `repot-temp.md`.
- La base de código nació de la plantilla del back de Balu (mismo esqueleto NestJS + pg + Redis; `repot-temp` aún se llama `bi-back-balu-board-v1`).
- Front consumidor: el contrato está en `FRONT.md`; el front de Fiduprevisora no está en esta carpeta de Analítica.
- Las convenciones de capas/skills se alinean con el harness `arness-back` (ver `arness-back.md`), aunque este repo usa su propia skill en `.claude/skills/`.
