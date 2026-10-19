# dev-back-bi-balu-recaudo-v1

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/dev-back-bi-balu-recaudo-v1`
> Nombre en `package.json`: `back-v2-balu` (v0.0.1)
> Cliente / producto: **Balú — Positiva** (Recaudo, Cuentas, Afiliaciones)
> Tipo: **Back-end (API REST de solo lectura)**

## Descripción / propósito

"Backend de analítica de Balu": nueva generación (v2) del back de reportes de Balú, de **solo lectura sobre PostgreSQL**, con NestJS 12, JWT y Redis como caché opcional. Construido sobre el **harness de agentes `@linktic/arness-back`** (repo `arness-back`), que fija stack, estructura, patrón por endpoint, seguridad y convenciones. A diferencia del back v1, **el login valida usuario y clave contra GAL** (proveedor de identidad de Positiva) y emite JWT propios de Balú.

## Stack y versiones

- Node.js **>= 24** (`.nvmrc`: v24.21.0), pnpm (vía corepack)
- **NestJS 12** en **ESM** (`@nestjs/common/core/config/jwt/platform-express`, `@nestjs/terminus` para health)
- `pg` 8.23 (sin ORM), `ioredis` 6, `nestjs-pino` + `pino-http` (logging), `class-validator`
- Jest (modo ESM con `--experimental-vm-modules`, `test/run-jest.mjs`), ESLint, Prettier
- Dev dependency `@linktic/arness-back` (instalada desde GitHub por SSH)

## Arquitectura y estructura de carpetas

```
src/
├── main.ts · app.module.ts · app.setup.ts (prefijo global /api/v1, excepto GET /health)
├── config/      # variables de entorno validadas al arrancar (environment.ts, configuration.ts, duration.ts)
├── common/      # auth (puerto IDENTITY_PROVIDER + adaptador GAL), database (multi-base, pool, errores), cache, http, logging, health
└── modules/
    └── positiva/                # <cliente>/<modulo>/<endpoint>
        ├── recaudo/daily-collection/
        ├── cuentas/settlement-contributors/
        └── afiliaciones/affiliates/
harness/         # AGENTS.md (reglas), agents/ (lider, implementador, revisor), playbooks/, decisiones/ (ADRs), reference/, init.sh
test/            # e2e: auth, health, cuentas, recaudo, afiliaciones
postman/         # colección + ambiente local
```

ADRs propios (`harness/decisiones/`): 0001 NestJS 12 en ESM con ESLint y Jest; 0002 paginación por cursor para listados grandes; 0003 login contra GAL.

## Funcionalidades / endpoints principales

| Método | Ruta | Descripción |
|---|---|---|
| POST | `/api/v1/auth/login` | `{ username, password, typeDocument?, typeUser? }` → reenvía a GAL; si GAL acepta, emite par JWT de Balú (`sub` = `<typeDocument>-<username>-<typeUser>`). 401 credenciales inválidas, 503 si GAL no responde |
| POST | `/api/v1/auth/refresh` | Renueva tokens sin volver a consultar GAL |
| GET | `/api/v1/cuentas/aportantes` | Aportantes de liquidación (settlement contributors) |
| GET | `/api/v1/afiliaciones/afiliados` | Listado de afiliados |
| GET | `/api/v1/recaudo/diario-por-cuenta` | Recaudo diario por cuenta |
| GET | `/health` | Healthcheck (terminus) |

Respuestas `{ data, meta? }`; guía de consumo en `FRONT.md` (refresh ante `401 TOKEN_EXPIRED`, reintento único).

## Fuentes de datos e integraciones

- **PostgreSQL** (configuración multi-base: `DATABASES` + prefijo `DB_MAIN_*`), solo lectura.
- **Redis** opcional (`CACHE_ENABLED`, `REDIS_URL`).
- **GAL** (login de Positiva, endpoint HTTPS configurado en `AUTH_LOGIN_URL`) mediante `fetch` nativo; nunca se devuelven ni loguean tokens o `user_info` de GAL.
- CI/CD: `azure-pipelines.yml` (init.sh --quick, lint, formato, build, tests unit + e2e, build Docker con `--ssh default` por la dependencia privada).

## Cómo ejecutarlo localmente

```bash
nvm use                      # Node 24
corepack enable
pnpm install                 # requiere acceso SSH a github.com/bi-analytics-linktic/harness-back
cp .env.example .env.local   # completar credenciales locales
./harness/init.sh            # verificación (harness, estructura, lint, formato, build, tests, /health)
pnpm start:dev
```

Scripts: `build`, `start`, `start:dev`, `start:debug`, `start:prod`, `lint`, `lint:check`, `format`, `format:check`, `test`, `test:watch`, `test:cov`, `test:debug`, `test:e2e`, `verify` (= `harness/init.sh`), `harness:status`, `harness:sync`.

Docker: `DOCKER_BUILDKIT=1 docker build --ssh default -t back-v2-balu .`

## Variables de entorno (solo nombres)

`PORT`, `LOG_LEVEL`, `DATABASES`, `DB_MAIN_HOST`, `DB_MAIN_PORT`, `DB_MAIN_NAME`, `DB_MAIN_USER`, `DB_MAIN_PASSWORD`, `DB_MAIN_SSL`, `DB_MAIN_POOL_MAX`, `DB_MAIN_STATEMENT_TIMEOUT_MS`, `CACHE_ENABLED`, `REDIS_URL`, `CACHE_DEFAULT_TTL_SECONDS`, `CACHE_KEY_PREFIX`, `CACHE_OP_TIMEOUT_MS`, `AUTH_LOGIN_URL`, `AUTH_LOGIN_TIMEOUT_MS`, `ACCESS_TOKEN` (ya no se lee; se mantiene porque `init.sh` lo exige), `JWT_ACCESS_SECRET`, `JWT_ACCESS_EXPIRES_IN`, `JWT_REFRESH_SECRET`, `JWT_REFRESH_EXPIRES_IN`.

## Pruebas

- ~59 archivos `*spec.ts` (unit en `src/`, e2e en `test/`: `auth`, `health`, `cuentas`, `recaudo`, `afiliaciones`).
- `pnpm test`, `pnpm test:e2e`, `pnpm test:cov`; verificación integral con `./harness/init.sh`.

## Estado git

- Remoto: `https://Linktic@dev.azure.com/Linktic/BI%20Analytics/_git/dev-back-bi-balu-recaudo-v1`
- Rama actual: `main` (única). Sin cambios salvo `package-lock.json` sin rastrear.
- Último commit: `94ec64a` (2026-10-05) «chore: merge initial azure devops readme into main».

```
94ec64a chore: merge initial azure devops readme into main
eda8002 fix(logging): redact secret query params in request urls
a50d3df feat(auth): validate login credentials against GAL
42ef4a3 docs: add project readme, frontend guide and postman collection
f4df169 ci: add docker image and azure pipeline
ba0943e chore(harness): add arness agent harness and tool bridges
8a98e9d test(e2e): add end-to-end suites for auth, health and positiva endpoints
137ee7e feat(app): wire application bootstrap and positiva client module
b651b85 feat(positiva/recaudo): add daily collection by account endpoint
07a803a feat(positiva/cuentas): add settlement contributors listing endpoint
```

## Notas

- Relación con `arness-back`: este repo es un consumidor del harness (`harness/.arness.json`, `pnpm harness:sync`). `CLAUDE.md` solo puentea a `harness/AGENTS.md`; también hay `GEMINI.md` y `.cursor/rules/agents.mdc`.
- Relación con `dev-back-bi-analytics-balu-dashboard-v1`: ambos sirven a Balú; el v1 tiene el tablero de Recaudo como módulo (`/recaudo/resumen|catalogos|matriz|sincronizacion`, login por token de rol) y este v2 expone endpoints distintos (`/recaudo/diario-por-cuenta`, etc.) con login GAL. No comparten código.
- Hay `package-lock.json` sin rastrear junto a `pnpm-lock.yaml` (probablemente un `npm install` accidental).
- Front asociado: tablero Balú (`https://front-bi-balu.linktic.com/`); no hay copia local del repo del Front.
