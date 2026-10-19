# poc-back-generacion-reportes

> Ruta local: `/Users/johnhernandez/Documents/Linktic/poc/poc-back-generacion-reportes`

## Descripción / propósito

Prueba de concepto (POC) de un **backend NestJS de reportería de solo lectura** sobre una o varias bases PostgreSQL, construida con el harness corporativo **`@linktic/arness-back`** (paquete instalado desde git: `bi-analytics-linktic/harness-back`). Es la base para el **generador dinámico de reportes**: un backend organizado por **cliente → módulo → endpoint**, en el que cada reporte es un `GET /api/v1/<modulo>/<segmento>`.

Hoy el proyecto está en estado **esqueleto**: solo existe la infraestructura (carga de entorno, módulo de base de datos multi-pool, instrumentación de NestJS Observe) y el `AppController` por defecto (`GET /` → "Hello World!"). Todavía no hay módulos de cliente en `src/modules/`.

Relación con otros proyectos:

- Es la **evolución de `backend-reporting-template`** (ver [backend-reporting-template.md](backend-reporting-template.md)): mismas ideas (multi-conexión `pg` por nombre lógico, SQL nativo parametrizado, capas controller/service/repository/sql/columns), pero empaquetadas en un harness versionado para agentes de IA y con NestJS 12.
- Su cliente objetivo natural es **Fiduprevisora** (el harness pone `src/modules/fiduprevisora/` como ejemplo). El contexto de la API BI existente de Fiduprevisora (`rdt-ms-reporter-bi`) está en `/Users/johnhernandez/Documents/md/Fidu` (`CLAUDE_FIDU.md`, `FRONT_FIDU.md`).
- El archivo `/Users/johnhernandez/Documents/Catálogo de integración para el generador dinámico de reportes.xlsx` (hojas LEEME, Modulos, Tablas, Campos, Relaciones, Queries) es la matriz de trabajo del equipo: dónde vive la información de cada módulo del CORE fiduciario, cómo se conecta (tabla local, `postgres_fdw`, etc.) y qué se puede cruzar. Es el insumo funcional de este generador.

## Stack y versiones

| Tema | Versión / elección |
|---|---|
| Runtime | Node.js (LTS), TypeScript `^6.0.2`, ESM (`"type": "module"`) |
| Framework | NestJS `^12.0.1` (`@nestjs/common`, `core`, `platform-express`) |
| BD | PostgreSQL vía `pg` `^8.23.1`, sin ORM |
| Observabilidad | `@nestjs/observe` `^0.1.8` (claves de ejemplo, sin configurar) |
| Despliegue | `@nestjs/mau` (dev) |
| Lint / formato | `oxlint` `^1.58`, Prettier `^3.4` |
| Tests | Vitest `^4.1.2` + Supertest, cobertura v8 |
| Gestor de paquetes | pnpm |
| Harness | `@linktic/arness-back` 0.1.0 |

Nota: el `harness/AGENTS.md` define como stack objetivo ESLint + Jest, `@nestjs/config`, `class-validator`, JWT, Redis (`ioredis`), `nestjs-pino` y `@nestjs/terminus`; el proyecto actual aún usa oxlint + Vitest y no tiene instaladas esas dependencias.

## Arquitectura y estructura de carpetas

```
src/
├── main.ts                 # bootstrap con ObserveInstrument; puerto PORT (3000 por defecto)
├── load-env.ts             # carga .env.local y .env con process.loadEnvFile
├── app.module.ts           # DatabaseModule.forRoot() + ObserveModule
├── app.controller.ts / app.service.ts   # Hello World por defecto
└── common/database/        # pools pg por nombre lógico (DATABASES=MAIN,...)
harness/                    # base del harness (no se edita): AGENTS.md, init.sh, agents/, skills/,
                            # playbooks/ (new-endpoint, new-module, add-database, review),
                            # reference/ (configuracion, contrato-http), decisiones/ (del repo)
scripts/check.sh            # wrapper no bloqueante de harness/init.sh (decisión 0001)
test/app.e2e-spec.ts
```

Puentes para agentes: `AGENTS.md` (symlink a `harness/AGENTS.md`), `CLAUDE.md`, `GEMINI.md`, `.cursor/`, `.claude/`, `.agents/`.

Patrón por endpoint definido en el harness (hexagonal por feature): `<e>.controller.ts`, `<e>.service.ts` (Cache-Aside + envelope `{ data, meta? }`), `<e>.repository.ts` (puerto + adaptador `pg`), `<e>.sql.ts` (funciones puras `(filters) => { text, values }`), `<e>.columns.ts` (contrato con el front), DTOs de query/filtros, entity y un spec por archivo.

Base de datos: `DatabaseModule` crea un `pg.Pool` por cada nombre listado en `DATABASES`; cada pool se registra con el token `PG_POOL_<NOMBRE>` y se abre en modo `default_transaction_read_only=on` con `statement_timeout` configurable y `application_name = arness-back`.

## Funcionalidades / endpoints

| Método | Ruta | Estado |
|---|---|---|
| GET | `/` | Hello World (plantilla de Nest) |

Planeado por el harness: `GET /api/v1/<modulo>/<segmento>` por reporte, `/health` (bases y caché), JWT access + refresh.

## Datos e integraciones

- Una o varias bases PostgreSQL definidas por convención `DB_<NOMBRE>_*` (en `.env.example`: `MAIN`).
- NestJS Observe (sin credenciales reales configuradas).

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env        # completar DB_MAIN_*
pnpm start:dev
pnpm check                  # verificación del harness, no bloqueante
```

## Variables de entorno (solo nombres)

`PORT`, `DATABASES`, y por cada base `<N>` listada: `DB_<N>_HOST`, `DB_<N>_PORT`, `DB_<N>_NAME`, `DB_<N>_USER`, `DB_<N>_PASSWORD`, `DB_<N>_SSL`, `DB_<N>_POOL_MAX`, `DB_<N>_STATEMENT_TIMEOUT_MS` (en el ejemplo, `N = MAIN`).

Existe un `.env` local (no versionar).

## Pruebas

- Unitarias: `pnpm test` (Vitest) — solo `src/app.controller.spec.ts`.
- E2E: `pnpm test:e2e` — `test/app.e2e-spec.ts` (`GET /`).
- Cobertura: `pnpm test:cov`. Lint: `pnpm lint` (oxlint).

## Estado git

- Repositorio git propio, rama `main`, **sin commits** y **sin remoto** configurado.
- Todo el contenido está sin seguimiento (`.agents/`, `.claude/`, `harness/`, `src/`, etc.).

## Notas

- Decisión `0001-verificacion-de-init-sh-no-bloqueante` (2026-10-08, John Hernández, estado *Propuesta*): `./scripts/check.sh` ejecuta `harness/init.sh` y siempre termina en 0, convirtiendo fallos en advertencias.
- `app.module.ts` tiene `appKey`/`appSecret` de Observe como marcadores de ejemplo; reemplazarlos por variables de entorno antes de usarlo en serio.
- Hay `dist/` y `tsconfig.build.tsbuildinfo` generados localmente.
