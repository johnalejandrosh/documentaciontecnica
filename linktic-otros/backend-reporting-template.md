# backend-reporting-template

> Ruta local: `/Users/johnhernandez/Documents/backend-reporting-template`

## Descripción / propósito

Plantilla base **NestJS + TypeScript + PostgreSQL (`pg`)** para proyectos de **reportería de solo lectura**. Define la arquitectura modular por dominio/reporte, la separación Controller / Service / Repository / SQL, la multi-conexión con `pg.Pool` nombrados y un contrato de respuesta estándar para el front:

```json
{ "count": 0, "module_name": "example", "report_name": "report", "query_id": "Q1", "data": [] }
```

### Relación con `poc-back-generacion-reportes`

Esta plantilla (septiembre 2026) es el **antecedente directo** de [poc-back-generacion-reportes](poc-back-generacion-reportes.md) (octubre 2026):

| Aspecto | backend-reporting-template | poc-back-generacion-reportes |
|---|---|---|
| NestJS | 11 | 12 (ESM) |
| Paquetes | npm | pnpm |
| Tests | Jest + ts-jest | Vitest (+ Supertest) |
| Lint | ESLint 9 + Prettier | oxlint + Prettier |
| Multi-conexión | `DB_<NOMBRE>_*` (REPORTING, ANALYTICS…) | `DATABASES` + `DB_<NOMBRE>_*` y pools en solo lectura |
| Reglas para agentes | `ARCHITECTURE.md`, `FRONT.md`, `mejoras/` | Harness `@linktic/arness-back` (`harness/AGENTS.md`, playbooks, decisiones) |
| Organización | `src/modules/<modulo>/<reporte>/` | `src/modules/<cliente>/<modulo>/<endpoint>/` |

Las mismas convenciones (archivo `.sql.ts` como único lugar con SQL, `.columns.ts` como contrato, envelope estándar) también son las de la API BI de Fiduprevisora (`rdt-ms-reporter-bi`, documentada en `/Users/johnhernandez/Documents/md/Fidu`).

## Stack y versiones

- NestJS `^11.0.0` (`common`, `core`, `platform-express`, `@nestjs/config ^4`, `@nestjs/helmet ^11`)
- `pg ^8.16.0`, `class-validator ^0.14.1`, `class-transformer ^0.5.1`, `rxjs ^7.8.1`
- TypeScript `^5.7`, ESLint `^9` + `typescript-eslint ^8`, `eslint-plugin-import`, Prettier `^3.6`
- Jest `^30` + ts-jest `^29.4`
- Gestor: npm

## Arquitectura y estructura de carpetas

```
src/
├── main.ts                       # prefijo API_PREFIX (api/v1), Helmet, ValidationPipe global, shutdown hooks
├── app.module.ts
├── infrastructure/database/      # database.config/service/module/types/constants (pools por nombre)
└── modules/
    ├── health/                   # GET /health → SELECT 1 en REPORTING
    └── example-report/           # controller, service, repository, sql (reporte de ejemplo)
test/app.e2e-spec.ts, jest-e2e.json
mejoras/README.md                 # registro de mejoras futuras (no implementar sin aprobación)
ARCHITECTURE.md                   # estructura objetivo por módulo, responsabilidades, SOLID, seguridad
FRONT.md                          # contrato de API para el front
```

Estructura objetivo por módulo (`ARCHITECTURE.md`): `<modulo>.module.ts`, `<modulo>.routes.ts`, `shared/` (tables, sql-builder, page-request DTO, cached-result) y por reporte `controller`, `service`, `repository`, `sql`, `columns`, DTOs de query y filtros, `entity` y specs.

## Funcionalidades / endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/v1/health` | Verifica la conexión `REPORTING` |
| GET | `/api/v1/example/report` | Reporte de ejemplo (`SELECT * FROM public.example_view`) con envelope estándar |

## Datos e integraciones

PostgreSQL con conexiones nombradas: `REPORTING` por defecto y `ANALYTICS` como ejemplo comentado.

## Cómo ejecutarlo localmente

```bash
cp .env.example .env
npm install
npm run start:dev
npm run lint && npm run format:check && npm test
```

## Variables de entorno (solo nombres)

`NODE_ENV`, `PORT`, `API_PREFIX`, `DB_REPORTING_HOST`, `DB_REPORTING_PORT`, `DB_REPORTING_USER`, `DB_REPORTING_PASSWORD`, `DB_REPORTING_NAME`, `DB_REPORTING_SSL`, `DB_REPORTING_POOL_MAX` (y opcionalmente las equivalentes `DB_ANALYTICS_*`).

## Pruebas

`npm test` (Jest), `npm run test:e2e` (`test/jest-e2e.json`), `npm run test:cov`, `npm run typecheck`.

## Estado git

No es un repositorio propio: la carpeta solo queda dentro del repositorio git del home del usuario (`/Users/johnhernandez`), sin commits. No tiene remoto propio. No tiene `node_modules` instalados.

## Notas

- `mejoras/` solo tiene el README que explica el procedimiento; no hay mejoras registradas.
- El README aclara que el proyecto no contiene credenciales reales.
