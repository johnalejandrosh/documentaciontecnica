# dev-back-bi-analytics-balu-dashboard-v1

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/dev-back-bi-analytics-balu-dashboard-v1`
> Nombre en `package.json`: `bi-back-balu-board-v1` (v0.0.1)
> Cliente / producto: **Balú — ARL Positiva** (tableros de Afiliaciones y Recaudo)
> Tipo: **Back-end (API REST de solo lectura)**

## Descripción / propósito

API **de solo lectura** que alimenta los tableros del aplicativo **Balú** (ARL Positiva). Hoy expone dos módulos independientes que comparten infraestructura (base de datos, caché, auth) pero no datos:

| Tablero | Fuente | Contrato que manda sobre el código | Roles |
|---|---|---|---|
| **Afiliaciones** | 5 tablas físicas `bi.rep_*` (`rep_affiliate`, `rep_affiliate_people`, `rep_employer`, `rep_sede`, `rep_divipola`) | `FRONT.md` | Afiliaciones / Gerencia |
| **Recaudo** (log bancario, desde 2026-09-08) | `bi.rep_log_bancario` | `RECAUDO.md` | Recaudo / Gerencia |

El servicio solo ejecuta `SELECT` contra una **réplica de lectura de PostgreSQL** (hot standby); la restricción está aplicada en varias capas (`src/database/sql-safety.ts`, `query-gate.ts`). Un tercer tablero, **Digiturno**, lee directamente de Supabase desde el Front y no usa este back (sí usa su login).

## Stack y versiones

- Node.js **>= 22** (Docker `node:22-alpine`), pnpm 11 (`packageManager: pnpm@11.22.0`)
- **NestJS 11** (`@nestjs/common/core/config/jwt/swagger/throttler`), TypeScript 5.7
- `pg` 8 (sin ORM), `ioredis` 5 (Redis, hoy deshabilitado por decisión de arquitectura), `helmet`, `class-validator`
- Jest 30 + Supertest (unit + e2e), ESLint 9 + Prettier

## Arquitectura y estructura de carpetas

```
src/
├── main.ts · app.module.ts · app.setup.ts · swagger.setup.ts
├── database/      # pool registry, multi-conexión, query-gate (control de admisión), sql-safety (solo SELECT)
├── cache/         # caché de dos niveles: nivel 1 en memoria del proceso + Redis (opcional), stale-while-revalidate
├── logs/          # registro de peticiones (verbo + URL) en logs/request.txt
├── common/        # auth, config (app.settings.ts = fuente única de parámetros), dataset en memoria, period, errores, métricas
└── modules/
    ├── affiliations/   # tablero Afiliaciones (+ dataset/ en memoria)
    ├── recaudo/        # tablero Recaudo (+ dataset/ en memoria)
    ├── auth/           # login por token de rol → JWT
    ├── health/
    ├── cache-admin/
    └── commands/       # CLI de consola (p. ej. bi-load-cache para precalentar caché)
context/       # DDL de referencia (Afiliaciones.sql, recaudo.sql)
docs/          # documentación técnica en español (api, arquitectura, base-de-datos, rendimiento, seguridad, configuracion) + docs/sql/
evolution/     # EVOLUTION.md: historia de cada evolutivo con mediciones
postman/       # colección y ambientes (local, remoto)
test/          # e2e (app, swagger)
```

Puntos clave de diseño (de `CLAUDE.md`):
- **Dataset en memoria**: copia columnar de las fuentes cargada al arrancar; `/resumen` y `/geo` (Afiliaciones) y los tres endpoints de datos de Recaudo se sirven en modo `memoria` (también existe modo `sombra`, que sirve SQL y compara fila a fila). Al ingresar un usuario se compara la última carga `EXITOSO` del ETL (`bi.log_actualizacion_reportes`) y, si cambió, recarga en segundo plano. Requiere tarea ECS de 2 GB.
- Una sola sincronización a la vez por base (`pg_try_advisory_lock`).
- Reglas SQL no negociables: `COUNT(*) FILTER`, `DISTINCT ON`, una sola referencia por CTE pesado, JSON ensamblado en TypeScript; ningún cast sobre columnas.
- Reintento ante conflicto de recuperación de la réplica (`40001`) → 503 explicativo.

## Funcionalidades / endpoints principales

Prefijo global `/api/v1` (salvo `/health`). Swagger en `/docs` solo con `nodeEnv: 'local'`. 17 endpoints documentados y auditados por `test/swagger.e2e-spec.ts`.

| Método | Ruta | Descripción |
|---|---|---|
| POST | `/auth/login` | Recibe un token de rol y emite par JWT (access ~15 min, refresh ~12 h con rotación) |
| POST | `/auth/refresh`, `/auth/logout` | Rotación / revocación |
| GET | `/auth/me` | Datos del usuario/rol |
| GET | `/afiliaciones/resumen` | KPIs del tablero (2 consultas o dataset en memoria) |
| GET | `/afiliaciones/geo` | Geografía (municipio por divipola, departamento por prefijo) |
| GET | `/afiliaciones/catalogos` | Catálogos para filtros |
| GET | `/afiliaciones/empresas`, `/afiliaciones/afiliados` | Listados paginados |
| GET | `/afiliaciones/exportacion` | Exportación |
| GET | `/afiliaciones/sincronizacion` | Estado real de los datos (caché/memoria/base, avance de carga) → indicador «Datos en vivo» del Front |
| GET | `/recaudo/resumen`, `/recaudo/catalogos`, `/recaudo/matriz`, `/recaudo/sincronizacion` | Tablero Recaudo (día de pago × cuenta; planillas y valor recaudado) |
| POST | `/cache/clear-cache` | Administración de caché |
| GET | `/health` | Healthcheck público |

## Fuentes de datos e integraciones

- **PostgreSQL (réplica de lectura, SSL obligatorio)**, esquema `bi`: tablas `rep_*` materializadas por un ETL externo que reconstruye cada tabla y la intercambia por `RENAME` (recrea sus índices). Tabla de control `bi.log_actualizacion_reportes`.
- **Redis** (opcional; deshabilitado, reemplazado por caché de nivel 1 en proceso).
- **AWS**: imagen en ECR (`dev-back-bi-balu-v1`) y despliegue en ECS (cluster `EcsDashboardsClusters`, servicio `dev-balu-svc-back`) vía `azure-pipelines.yml` (trigger en `main`, variable group `bi-balu-back-secrets`).

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env     # completar credenciales de la réplica y secretos
pnpm run start:dev       # http://localhost:3001  (Swagger /docs con NODE_ENV local)
```

Scripts: `build`, `start`, `dev` / `start:dev`, `start:debug`, `start:prod`, `lint`, `format`, `cli` / `cli:prod` (comandos de consola, p. ej. precalentar caché), `test`, `test:watch`, `test:cov`, `test:e2e`. Docker multi-stage (`Dockerfile`, usuario sin privilegios). Colección Postman en `postman/`.

## Variables de entorno (solo nombres)

`.env.example`: `PORT`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `TOKEN_DIGITURNO`, `TOKEN_AFILIACIONES`, `TOKEN_RECAUDO`, `TOKEN_GERENCIA`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `REDIS_HOST`, `REDIS_PASSWORD`.

El resto de parámetros (esquema, SSL, `work_mem`, timeouts, pool, modo del dataset, TTLs de caché) viven ahora en `src/common/config/app.settings.ts` (un `SHEMA` en `.env` se ignora). Existen además `.env` y `.env.local` locales (no documentar valores).

## Pruebas

- ~55 archivos `*.spec.ts` (unit) en `src/` + e2e en `test/` (`app.e2e-spec.ts`, `swagger.e2e-spec.ts`).
- Tests de forma de SQL (p. ej. `affiliations.sql.spec.ts`, «ninguna columna de fecha lleva cast»), de seguridad SQL, query-gate, caché, dataset en memoria.
- `pnpm test`, `pnpm test:e2e`, `pnpm test:cov`.

## Estado git

- Remoto: `https://Linktic@dev.azure.com/Linktic/BI%20Analytics/_git/dev-back-bi-analytics-balu-dashboard-v1`
- Rama actual: `main` (limpia). Otras ramas: `feature/best-structure-affiliations`, `feature/swagger`, `fix/consumo_data`, `fix/revert`; remota `feat/dataset-modo-memoria`.
- Último commit: `ef24433` (2026-10-08) «Recaudo con el modelo de actualización de Afiliaciones, sirviendo de memoria».

Últimos commits:
```
ef24433 Recaudo con el modelo de actualización de Afiliaciones, sirviendo de memoria
b08e47b Dataset en memoria sirviendo de memoria: ajuste, test y documentación
8816ce5 Estado real de los datos: GET /afiliaciones/sincronizacion y turno de carga entre procesos
c044fce alineación de repos
df8c82c actualización de documentación
4abcb59 Merged PR 232931: Dataset en memoria encendido en modo sombra
946985a Dataset en memoria encendido en modo sombra: test y documentación
fa0abb2 Merged PR 232901: Fix/consumo data
ab3f23d Merge origin/main en fix/consumo_data
2264228 ajuste variables de entorno
```

## Notas

- **Front asociado**: el tablero Balú (Front del equipo BI Analytics) publicado en `https://front-bi-balu.linktic.com/`; consume la API vía `NEXT_PUBLIC_AFILIACIONES_API_URL`. **No se encontró copia local del repositorio del Front** en el equipo.
- **Contexto externo**: `/Users/johnhernandez/Documents/README_BALU.md` es una copia antigua del README de este repo (cita `docs/es/`, que ya no existe), y `/Users/johnhernandez/Documents/md/Balu/CLAUDE_BALU_AFILIACIONES.md` y `FRONT_BALU_AFILIACIONES.md` son copias anteriores (agosto 2026) de su `CLAUDE.md` y `FRONT.md`.
- **POC relacionada**: `poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1` es un clon del mismo repo en la rama `feature/poc-duck-parker` (motor DuckDB sobre Parquet). Ver su documento.
- **Sucesor/variante**: `dev-back-bi-balu-recaudo-v1` (`back-v2-balu`) es un back nuevo de Balú sobre NestJS 12 con login contra GAL.
- Pendientes de base (fuera del repo, equipo ETL): `SET STATISTICS 500` y `VACUUM` programado sobre las tablas `rep_*`.
