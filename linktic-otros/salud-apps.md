# salud-apps (Health Apps)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/projectos_alter/salud-apps`
> Repositorio: `bi-analytics-linktic/dev-healthapps-v2`

## Descripción / propósito

**Health Apps** es un centro de monitoreo centralizado del estado, la disponibilidad y el rendimiento de las aplicaciones y servicios en operación. Muestra en tarjetas, indicadores y gráficas qué servicios están **operativos, degradados o fuera de servicio**, con uptime, tiempos de respuesta, última verificación e historial de estados, y alertas ante caídas o degradación. Es una aplicación web **mobile-first** (el equipo la consulta desde el móvil durante incidentes), responsive para escritorio, con tema claro y oscuro.

El conocimiento del proyecto (negocio, producto, arquitectura, roadmap, ADR y Work Items) vive en `knowledge/`, generado con **kaddo** (`.kaddo/`).

## Stack y versiones

| Capa | Tecnología |
|---|---|
| Framework | Next.js `16.3.8` (App Router, `proxy.ts`), React `19.2.8`, TypeScript estricto |
| UI | Tailwind CSS 4, shadcn `^4.21` + Radix UI, lucide-react, motion, next-themes, react-hot-toast, canvas-confetti |
| Gráficas | Chart.js `^4.5` + react-chartjs-2 |
| Persistencia | PostgreSQL 18, Prisma `7.10.0` + `@prisma/adapter-pg`, `pg ^8.23` |
| Auth | `jose` (JWT), `@node-rs/argon2` (hash), sesiones revocables en BD, cookies `httpOnly` |
| Validación | `zod ^4.6` |
| Worker | `tsx` (proceso aparte `src/worker/index.ts`) |
| Tests | Vitest `^5`, Testing Library, jsdom |
| Runtime | Node.js >= 24 (`.nvmrc`), pnpm 11.20.0 |
| Contenedores | Dockerfile multi-stage (`base`, `deps`, `dev`, `builder`, `migrator`, `worker`, `prod`), docker-compose para desarrollo |
| Despliegue | Vercel (modo manual de verificaciones) o contenedores; base en Supabase (esquema `app_health`) |

## Arquitectura y estructura de carpetas

```
app/                 rutas App Router
  (auth)/            login, registro
  (app)/             panel (/), servicios (lista, nuevo, [id], [id]/editar), alertas, usuarios/nuevo
  api/               Route Handlers delgados que delegan en src/features/<feature>/server/
src/
  components/        UI reutilizable sin negocio
  features/          auth, checks, dashboard, health-status, help, service-catalog,
                     service-detail, uptime, users
  layouts/ hooks/ lib/ (incluye lib/db/config.ts) types/ generated/ (cliente Prisma) test/
  worker/index.ts    verificaciones periódicas (FOR UPDATE SKIP LOCKED)
prisma/              schema.prisma, migrations/ (8), seed.ts
prisma.config.ts     bloquea migrate dev/reset/db push/execute contra hosts no locales
scripts/             db-migrate.sh, supabase/app-health-setup.sql
docker/postgres/     init de la base local
knowledge/           business, product, tech (codebase, current-state, decisions ADR-001..006), delivery (roadmap, work-items)
resources/lineagrafica/  línea gráfica
```

Decisiones (ADR en `knowledge/tech/decisions/`): 001 persistencia PostgreSQL + Prisma; 002 JWT + sesiones revocables; 003 roles (`superadmin`, `admin`, `nuevo`) y permisos `<recurso>.<acción>` revocables por usuario; 004 worker de verificaciones; 005 historial y retención (resultados 30 días, cambios de estado 365 días); 006 canales de alerta (borrador: bandeja en la app + webhook Slack/Teams).

## Funcionalidades / endpoints / pantallas

Pantallas: `/login`, `/registro`, `/` (panel), `/servicios`, `/servicios/nuevo`, `/servicios/[id]`, `/servicios/[id]/editar`, `/alertas`, `/usuarios/nuevo`.

API (`app/api/`):

| Ruta | Uso |
|---|---|
| `/api/auth/login`, `register`, `refresh`, `logout`, `logout-all`, `me`, `session`, `change-password` | Autenticación y sesión |
| `/api/services`, `/api/services/[id]`, `/api/services/[id]/detail` | Catálogo de servicios monitoreados |
| `/api/dashboard` | Datos del panel |
| `/api/checks/run` | Verificación manual inmediata (permiso `services.read`, `maxDuration` 60 s) |
| `/api/users` | Usuarios |
| `/api/health` | Salud de la propia app |

Modelo Prisma: `Role`, `Permission`, `RolePermission`, `User`, `UserRevokedPermission`, `UserSession`, `Service`, `ServiceCheck`, `ServiceStatusChange`; enums `HealthStatus`, `CheckOutcome`.

Modos de verificación: `worker` (automático, Docker/contenedores) o `manual` (en Vercel, botón **Verificar ahora**); se fuerza con `CHECKS_MODE`.

## Datos e integraciones

- PostgreSQL: local (docker-compose, base `health_apps`, esquema `app_health`) o Supabase (esquema `app_health`, rol `app_health_owner`; en Vercel, Transaction pooler puerto 6543). No usa `supabase-js`.
- Verificaciones HTTP a las URLs de los servicios registrados; `MONITOR_CANARY_URL` opcional para distinguir caídas propias.

## Cómo ejecutarlo localmente

```bash
cp .env.example .env
docker compose up               # app (next dev) + worker + PostgreSQL 18; aplica migraciones y seed
# o fuera de Docker:
docker compose up db
pnpm install && pnpm dev        # http://localhost:3000
pnpm worker:dev                 # worker en otra terminal
pnpm db:migrate                 # solo base local
pnpm db:seed                    # roles, permisos y primer superadmin
```

Producción: `docker build --target prod`, `--target migrator` (migraciones) y `--target worker`.

## Variables de entorno (solo nombres)

- Base: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_SCHEMA`, `DB_SSLMODE`, `DB_POOL_MAX`, `DATABASE_URL` (opcional), `LOCAL_DB_PORT` (compose)
- Auth: `JWT_ACCESS_SECRET`, `JWT_ACCESS_TTL_MINUTES`, `REFRESH_TOKEN_TTL_MINUTES`, `AUTH_COOKIE_SECURE`
- Seed: `SEED_SUPERADMIN_EMAIL`, `SEED_SUPERADMIN_USERNAME`, `SEED_SUPERADMIN_PASSWORD`
- Worker: `CHECKS_MODE`, `CHECK_TICK_SECONDS`, `CHECK_CONCURRENCY`, `MONITOR_CANARY_URL`, `CHECK_RESULTS_RETENTION_DAYS`, `STATUS_HISTORY_RETENTION_DAYS`, `CHECK_LOCALHOST_ALIAS`
- Plataforma: `VERCEL` (la inyecta Vercel)

Hay `.env` y un `.envhhh` locales (no versionar; revisar si `.envhhh` sobra).

## Pruebas

- `pnpm test`: unitarias con Vitest (~37 archivos `*.test.*` en `src/`).
- `pnpm test:integration`: contra la base del `.env` (crea y borra datos; usar solo con la base local).
- `pnpm lint`, `pnpm typecheck`.

## Estado git

- Remoto: `origin` → `https://github.com/bi-analytics-linktic/dev-healthapps-v2.git`
- Rama actual: `fix/db-pool-vercel` (árbol limpio)
- Últimos commits:
  - `6e1d53a` fix: limita el pool de conexiones a la base en Vercel
  - `3d35e43` Merge PR #2 (fix/prisma-generate-build)
  - `f525709` fix: genera el cliente de Prisma antes de next build
  - `f78513f` Merge PR #1 (chore/comentario-readme)
  - `de4b5c8` chore: agrega comentario de migración en README
  - `79e86a5` ajuste
  - `7802adc` first commit

## Notas

- El README indica que el repositorio fue migrado a `bi-analytics-linktic/dev-healthapps-v2`.
- Work items: 22 completados y 2 en borrador; pendientes abiertos: valores del tema claro, canal de alertas (ADR-006), contrato de envío de métricas (WI-018), límite de intentos de login.
- En Next 16 el middleware es `proxy.ts`; `params` y `searchParams` son promesas.
