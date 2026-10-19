# repot-temp

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/repot-temp`
> Tipo: **back (copia temporal / plantilla)** · Cliente: **Fiduprevisora** (remoto en "384 - CORE FIDUPREVISORA"), base derivada de Balu

## Descripción / propósito

Copia local temporal del repositorio `rdt_ms_api_erp_report_generator` posicionada en el **commit inicial** (`51cb183`). Contiene la base común de un servicio BI de solo lectura (NestJS + `pg` + Redis): configuración, capa de datos multi-pool, caché, logs de peticiones y healthcheck, más un único módulo de negocio: **terceros** de Fiduprevisora (`GET /api/v1/thirdparties`).

Su `package.json` todavía se llama `bi-back-balu-board-v1` y su `CLAUDE.md` lo describe como "base común de servicios BI del aplicativo Balú": es la plantilla heredada del back de Balu que se usó para arrancar el reporter de Fiduprevisora.

## Stack y versiones

Idéntico a `rdt-ms-reporter-bi` (el `package.json` solo difiere en el nombre):
Node >= 22, pnpm 11, NestJS 11, `pg` 8 (sin ORM), `ioredis` 5, `helmet`, `class-validator`, Swagger, Throttler, TypeScript 5.7, Jest 30.

## Arquitectura y estructura de carpetas

```
src/
├── common/      # config (EnvConfigService, db-schema), filtros, contexto, errores
├── database/    # pool factory/registry, query-gate, sql-safety
├── cache/       # Redis + claves
├── logs/        # request.txt
└── modules/
    ├── health/
    ├── cache-admin/
    └── fiduprevisora/thirdparties/   # controller, service, repository, sql, columns, tables, types
test/            # e2e
docs/es/         # misma documentación técnica que el reporter
QUERY-THIRD-PARTIES.md   # SQL/explicación del reporte de terceros
```

## Funcionalidades / endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Estado del servicio, BD y Redis |
| POST | `/api/v1/cache/clear-cache` | Limpia caché Redis |
| GET | `/api/v1/thirdparties` | Listado maestro de terceros (paginado `page`/`pageSize`) |

Sin autenticación; solo lectura.

## Fuentes de datos e integraciones

- PostgreSQL (una sola conexión `DB_*`, esquema configurable `SHEMA`, default `bi`).
- Redis opcional.
- `azure-pipelines.yml` heredado de Balu: "Release Backend BI Balu Dashboard v1" (build → docker → push a ECR `dev-back-bi-balu-v1` → update de servicio ECS `dev-balu-svc-back` en `EcsDashboardsClusters`), con variable group `bi-balu-back-secrets`. Es decir, el pipeline **no** corresponde a Fiduprevisora en esta versión.

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env
pnpm run dev            # puerto 3001
```
No hay `.env` real en el directorio, solo `.env.example`.

## Variables de entorno (solo nombres)

`NODE_ENV`, `PORT`, `CORS_ORIGINS`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSL`, `SHEMA`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_DB`, `CACHE_ENABLED`, `CACHE_KEY_PREFIX`, `CACHE_TTL_SECONDS`, `REQUEST_LOG_ENABLED`, `REQUEST_LOG_FILE`, `REQUEST_LOG_MAX_BYTES`.

## Pruebas

`pnpm run test` / `test:e2e` / `test:cov`. 14 archivos `*.spec.ts` (sql-safety, query-gate, cache, logs, db-schema, health, thirdparties).

## Estado git

- Remoto: `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_report_generator` (el mismo que `rdt-ms-reporter-bi`)
- Rama actual: `revert/initialcommit` (locales: `main`, `dev`, `revert/51cb183...`; remotas: `main`, `dev`, `feature/back-fidu`, `feature/reports`, `feature/reports-materialize-view`)
- HEAD: `51cb183` (2026-08-27) "Commit inicial" — único commit de la rama.
- El repo tiene más historia en otras ramas (p. ej. `8e79320` "Implementacion del modulo de bank-reconciliation", PRs 219341/219212, commits de pipeline "Start bases on azure pipeline").
- Árbol de trabajo limpio.

## Notas

- **Comparación con `rdt-ms-reporter-bi`**: mismo remoto, misma raíz de historia (`51cb183` es el primer commit en ambos). `repot-temp` = instantánea del arranque (1 módulo, 3 endpoints, 14 specs); `rdt-ms-reporter-bi` = versión activa (15 submódulos, ~79 reportes, multi-conexión, ~545 specs, pipeline propio a ECR de Fidu). Todo el desarrollo nuevo debe hacerse en `rdt-ms-reporter-bi`.
- Por el nombre de la rama (`revert/initialcommit`) parece haberse usado para revisar o revertir al estado inicial; candidato a eliminar una vez confirmado que no tiene trabajo pendiente.
