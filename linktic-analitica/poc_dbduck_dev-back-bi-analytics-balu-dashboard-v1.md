# poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1`
> Nombre en `package.json`: `bi-back-balu-board-v1`
> Cliente / producto: **Balú — ARL Positiva** (Afiliaciones / Recaudo)
> Tipo: **POC de back-end** (motor analítico alternativo)

## Descripción / propósito

Prueba de concepto sobre el back del tablero Balú: **sustituir la agregación en la réplica PostgreSQL por DuckDB leyendo instantáneas Parquet**. Regla rectora: los endpoints deben devolver **exactamente la misma respuesta** (mismos campos, cifras y orden); solo cambia de dónde salen las filas.

Diseño (`evolution/ESPECIFICACION-DUCKDB-PARQUET.md`):
- Un **exportador único** (fuera del API) lee PostgreSQL una vez por ciclo y publica **instantáneas completas y versionadas** en S3 (`manifest.json` + un `.parquet` por tabla, puntero `current.json`).
- Cada tarea del API descarga la instantánea, la abre con **DuckDB** y la sustituye de forma atómica; los repositorios ejecutan el **mismo SQL** contra DuckDB.
- Interruptor `ANALYTICS_ENGINE` (apagado por defecto → PostgreSQL).

Estado según la especificación: **implementada en código (fases 1–3), apagada por defecto**; arnés de equivalencia con **206 de 206 casos idénticos** contra PostgreSQL 14 local con datos sintéticos. Falta la Fase 0 contra la réplica real e infraestructura (bucket, IAM, tarea programada).

### Diferencia con `dev-back-bi-analytics-balu-dashboard-v1`

- **Mismo remoto** de Azure DevOps (`dev-back-bi-analytics-balu-dashboard-v1`), pero es un **clon independiente** en la rama local `feature/poc-duck-parker`, que **no existe en el remoto** (no está publicada).
- Parte de un punto anterior de `main` (último commit `06e0aa7`, 2026-10-06, tras «Cache de dos niveles…»), es decir, **no incluye** el dataset en memoria ni `/sincronizacion` que tiene hoy el repo principal (`ef24433`).
- **Todo el trabajo DuckDB está sin commitear**: ~204 entradas en `git status` (archivos modificados + nuevos como `src/analytics/`, comandos `bi-export-parquet`, `bi-compare-engines`, `bi-load-cache-acotado`, `consistent-read.service.ts`, `storage/`).
- Dependencias añadidas respecto al repo principal: `@duckdb/node-api` 1.5.6-r.1 y `@aws-sdk/client-s3` 3.x.

## Stack y versiones

Igual que el back principal (Node >= 22, pnpm 11, NestJS 11, `pg` 8, `ioredis` 5, Jest 30) más:
- **DuckDB** (`@duckdb/node-api` 1.5.6-r.1)
- **AWS SDK S3** (`@aws-sdk/client-s3`) para publicar/leer instantáneas
- Formato **Parquet**

## Arquitectura y estructura de carpetas

Misma estructura que el back principal (`src/database`, `src/cache`, `src/common`, `src/logs`, `src/modules/{affiliations,recaudo,auth,health,cache-admin,commands}`), más:

```
src/analytics/      # duckdb-database.service, engine-router (PG ↔ DuckDB), snapshot-manager/store/manifest, duckdb-values
src/modules/commands/
  ├── bi-export-parquet.*      # exportador PG → Parquet (+ validación y progreso)
  ├── bi-compare-engines.*     # arnés de equivalencia PG vs DuckDB (casos + diff)
  └── bi-load-cache-acotado.*  # precalentado de caché acotado
src/database/consistent-read.service.ts
storage/local/<snapshotId>/*.parquet        # instantánea local
storage/s3/{current.json,snapshots/<id>/}   # simulación local del bucket S3
docs/sql/90-datos-sinteticos-duckdb.sql     # datos sintéticos para pruebas
evolution/ESPECIFICACION-DUCKDB-PARQUET.md
```

Instantánea presente en disco: `20261007T072311Z` con `rep_affiliate`, `rep_affiliate_people`, `rep_employer`, `rep_divipola`, `rep_log_bancario`.

## Funcionalidades / endpoints principales

Mismos endpoints de la versión de `main` de la que parte: `/api/v1/auth/*`, `/api/v1/afiliaciones/{resumen,geo,catalogos,empresas,afiliados,exportacion}`, `/api/v1/recaudo/*`, `/api/v1/cache/clear-cache`, `/health`. Lo nuevo son **comandos de consola** (`pnpm cli ...`): exportar Parquet, comparar motores, precalentar caché acotado.

## Fuentes de datos e integraciones

- PostgreSQL (réplica de lectura, esquema `bi`) como fuente del exportador y motor por defecto.
- **DuckDB embebido** sobre Parquet como motor alternativo.
- **S3** para instantáneas versionadas (simulado localmente en `storage/s3/`).
- Redis opcional; despliegue ECR/ECS vía `azure-pipelines.yml` (último commit es una actualización del pipeline).

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env        # completar credenciales; ANALYTICS_ENGINE para elegir motor
pnpm run start:dev
pnpm cli <comando>          # p. ej. exportar Parquet o comparar motores
```

Scripts: idénticos al back principal (`build`, `start:dev`, `start:prod`, `cli`, `cli:prod`, `lint`, `format`, `test`, `test:e2e`, `test:cov`).

## Variables de entorno (solo nombres)

`NODE_ENV`, `PORT`, `TRUST_PROXY`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSL`, `SHEMA`, `DB_STATEMENT_TIMEOUT_MS`, `DB_QUERY_TIMEOUT_MS`, `DB_POOL_MAX`, `DB_CONNECT_TIMEOUT_MS`, `DB_MAX_CONCURRENT_QUERIES`, `DB_ADMISSION_TIMEOUT_MS`, `DB_WORK_MEM`, `DB_CONNECTIONS`, `TOKEN_DIGITURNO`, `TOKEN_AFILIACIONES`, `TOKEN_RECAUDO`, `TOKEN_GERENCIA`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `JWT_ACCESS_EXPIRES_IN`, `JWT_REFRESH_EXPIRES_IN`, `CORS_ORIGINS`, `CACHE_MEMORY_ENABLED`, `CACHE_MEMORY_MAX_ENTRIES`, `CACHE_MEMORY_MAX_MB`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_DB`, `REDIS_PASSWORD`, `CACHE_ENABLED`, `CACHE_TTL_SECONDS`, `CACHE_KEY_PREFIX`, `CACHE_STALE_TTL_SECONDS`, `REQUEST_LOG_ENABLED`, `REQUEST_LOG_FILE`, `REQUEST_LOG_BASE_URL`, `REQUEST_LOG_MAX_BYTES`, **`ANALYTICS_ENGINE`**, **`ANALYTICS_DUCKDB_COLLATION`**.

Archivos locales: `.env`, `.env.local`, `.env.local.v1` (no versionado).

## Pruebas

- Unit (`*.spec.ts`) incluidos los nuevos: `duckdb-database.service.spec`, `engine-router.spec`, `snapshot-*.spec`, `bi-compare-engines.spec`, `bi-export-parquet.*.spec`, `analytics.config.spec`.
- Arnés de equivalencia `bi-compare-engines`: 206/206 casos idénticos (datos sintéticos).
- e2e en `test/` (`app`, `swagger`).

## Estado git

- Remoto: `https://Linktic@dev.azure.com/Linktic/BI%20Analytics/_git/dev-back-bi-analytics-balu-dashboard-v1` (mismo que el back principal)
- Rama actual: `feature/poc-duck-parker` (solo local). Otras locales: `feature/best-structure-affiliations`, `feature/swagger`, `fix/revert`, `main`.
- Último commit: `06e0aa7` (2026-10-06) «Update azure-pipelines.yml for Azure Pipelines».
- **Árbol de trabajo con muchos cambios sin commitear** (~204 entradas: modificados + nuevos).

```
06e0aa7 Update azure-pipelines.yml for Azure Pipelines
f98f557 Merged PR 225234: Cache de dos niveles: nivel 1 en memoria del proceso y TTL por familia
cf9bfbb Cache de dos niveles: nivel 1 en memoria del proceso y TTL por familia
ff774cc Merged PR 223881: Ajuste de Cache
0e1be5f Ajuste de Cache
c950c07 Merged PR 223851: Ajuste de Cache
9ddd3f4 Ajuste de Cache
b8c7fee Merged PR 223093: Fix/data refactory
d1a70ce Medicion contra la replica, reparacion de indices y limpieza de codigo muerto
f148efb Rendimiento Back-Front, documentacion verificada e historia unificada
```

## Notas

- Riesgo: el trabajo de la POC solo existe en esta copia local (rama no publicada y cambios sin commitear).
- El repo principal tomó otro camino para aliviar la réplica: **dataset en memoria** dentro del proceso (en `main`), en lugar de DuckDB/Parquet.
- `storage/` contiene archivos Parquet con datos de las fuentes; tratarlos como datos sensibles (datos personales de afiliados) y no compartir.
- Front asociado: el mismo tablero Balú (`https://front-bi-balu.linktic.com/`), sin copia local.
