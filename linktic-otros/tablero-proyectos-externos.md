# Tablero Proyectos Externos (01_tablero)

> Ruta local del paquete: `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/01_tablero`
> Código: `.../01_tablero/repositorio/tablero-proyectos-externos`
> Repositorio: `bi-analytics-linktic/dev-bi-proyectosexternos`

## Descripción / propósito

Aplicación web con los **tableros de gestión de mesa de ayuda** de los proyectos externos de LinkTIC: **BALÚ** (Positiva ARL), **SGDEA / Docum**, **Fiscalía**, **UGPP**, **La Equidad**, **CRM / Ágora CRM (Positiva)**, **Core Positiva**, **Rama Judicial**, **Cymetria (página web Positiva)** y **Tu Negocio en Línea (TNL)**. Muestra volumen de tickets, estados, cumplimiento de ANS, canales, tipología, categorías, patrones temporales, desempeño de analistas y satisfacción, e incluye **análisis automático con IA** (OpenAI) que sirve de insumo para los informes mensuales.

Forma parte del **paquete de entrega del 13 de octubre de 2026**: el equipo de proyectos internos (Innovación y Transformación Digital) lo entrega al equipo de proyectos externos para que lo opere en su propia infraestructura. El paquete completo está en `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/` con:

- `README.md` (qué trae el paquete y orden de instalación), `PENDIENTES.md` (seguridad, configuración, usuarios, preguntas abiertas) y `README_MODULO_BALU.md` (referencia del módulo BALÚ, ANS 13 y ANS 18, al 19-oct-2026).
- `01_tablero/` (este documento), [`02_etl_data_core/`](etl-data-core.md), y además en la carpeta: [`supabase-local/`](supabase-local.md) y [`requerimientostablerossoporte/`](requerimientos-tableros-soporte.md).

## Stack y versiones

| Capa | Tecnología |
|---|---|
| Framework | Next.js `16.2.2` (App Router, Server Components, Server Actions, React Compiler `babel-plugin-react-compiler 1.0.0`) |
| UI | React `19.2.4`, Tailwind CSS 4, Radix Select, lucide-react, framer-motion, next-themes |
| Gráficos | Recharts `^3.8` |
| Datos | PostgreSQL (esquema `tablero_proyectos_externos`) vía `pg ^8.23` (desde 15-oct-2026); antes `@supabase/supabase-js ^2.101` |
| Auth | Correo + contraseña (hash scrypt, cookie firmada `tpe_session`) desde 15-oct-2026; la versión entregada usaba Supabase Auth con Google OAuth |
| IA | `openai ^6.33` (`gpt-4o-mini`) |
| Importación / exportación | `xlsx 0.18.5`, `html-to-image`, `jspdf ^4.2` |
| Lenguaje / gestor | TypeScript 5, npm (`package-lock.json`); versión app `0.1.55` |

## Arquitectura y estructura de carpetas

```
app/
  page.tsx, login/, admin/ (usuarios), admin/modulos/
  [moduleCode]/             tablero genérico por código de módulo
  balu/ sgdea/ fiscalia/ ugpp/ equidad/ core-positiva/ crm-positiva/ agora-crm/
  crm-positiva-agora/ (redirige a /agora-crm) rama-judicial/ pagina-web-positiva-cymetria/ tu-negocio-en-linea/
  actions/                  Server Actions: balu.ts (agregados), data.ts, modules.ts, users.ts
  api/ai-analysis-general/  análisis IA (POST) · api/ai-analysis/ (legado)
components/                 admin, auth, shared, ui
lib/                        db.ts, session.ts, auth.actions.ts, auth.guard.ts, password.ts, brands.ts,
                            allowed-domains.ts, rollup-aggregator.ts, *-analytics.ts (core, crm, equidad, ugpp),
                            project-legends.ts (ANS por proyecto), ai-analysis/ (prompts, ans-contractuales, informe-mensual)
supabase/migrations/        14 migraciones SQL (rollups, RPC dashboard, serie unificada, login con contraseña)
scripts/                    clear-data, set-password, unificado-refresh/diagnostico/comparar
docs/                       login-y-base-sin-supabase.md, diagnósticos SQL
proxy.ts                    (middleware de Next 16) protección de rutas, excluye /api/*
```

Documentación del paquete en `01_tablero/`: `documentacion-tecnica.md`, `arquitectura.md` (diagramas mermaid: contexto, login, importación, serie unificada, IA, modelo de datos, despliegue), `especificacion-apis.md`, `base-de-datos.md` (DDL completo + orden de carga), `runbook-cargue-excel.md`.

## Funcionalidades / endpoints / pantallas

- **Tableros por módulo** con filtros y rango de fechas; datos leídos de rollups precalculados vía RPC `dashboard_rollup` / `dashboard_rollup_unificado(p_module_id, p_desde, p_hasta)`.
- **Importación de Excel** por administradores (Server Action, `bodySizeLimit` 100 MB) y reconstrucción del rollup.
- **Análisis con IA** (`POST /api/ai-analysis-general`) y ANS contractuales calculados en código.
- **Exportación** a imagen, PDF y Excel.
- **Administración** de usuarios (roles `admin` / `user`, asignación de módulos, asignación/reset de contraseña) y de módulos.
- Incrustable en Zendesk de 3T (`frame-ancestors https://3tcapital-1.zendesk.com`).

## Datos e integraciones

```
Excel de cada proyecto ──(carga desde la app)──► data ────────────────┐
                                                                      ├─► rollups ──► tablero
API Data Core ──(ETL, cada hora)──► data_core_cases ──► data_unificado ┘
```

- **BALÚ, Docum y Fiscalía**: Excel hasta julio 2026 y API Data Core desde el 1-ago-2026 (corte en hora Bogotá, configurado en `modulo_fuente_api`).
- Resto de proyectos (incluida Rama Judicial): 100 % Excel.
- Tablas principales: `modules`, `profiles`, `profile_modules`, `data`, `data_core_cases`, `modulo_fuente_api`, `data_rollup*`, `data_rollup_unificado*`, vista `data_unificado`.
- Datos iniciales en `01_tablero/datos/` (módulos, 29 perfiles, 119 asignaciones, configuración de API y `_data__` de ~313 MB). **Contienen datos personales**: acceso restringido, no subir a repositorios.
- OpenAI para el análisis.

## Cómo ejecutarlo localmente

```bash
cd 01_tablero/repositorio/tablero-proyectos-externos
npm install
cp .env.example .env                      # DATABASE_URL, DB_SCHEMA, AUTH_SECRET, OPENAI_API_KEY...
psql "$DATABASE_URL" -f supabase/migrations/20261015_01_login_con_contrasena.sql   # una vez por base
node scripts/set-password.mjs <correo-admin>   # primer administrador
npm run dev                               # http://localhost:3000
```

Base local: ver [supabase-local.md](supabase-local.md) (Postgres en `127.0.0.1:54322`). Instalación completa desde cero: `base-de-datos.md` secciones 1 a 8.

## Variables de entorno (solo nombres)

`DATABASE_URL`, `DB_SCHEMA`, `DB_POOL_MAX`, `AUTH_SECRET`, `NEXT_PUBLIC_BASE_URL`, `NEXT_PUBLIC_FORCE_BRAND`, `OPENAI_API_KEY`.

Hay varios archivos locales con valores (`.env`, `.env.local-db`, `.env.supabase-backup`): no versionar ni compartir.

## Pruebas

No hay pruebas automatizadas. Verificación manual con la checklist de la sección 6 del README de la entrega y los scripts de diagnóstico (`scripts/unificado-diagnostico.mjs`, `docs/*.sql`).

## Estado git

- Repositorio inicializado sobre la copia del código (la entrega se hizo sin historial).
- Remoto: `origin` → `https://github.com/bi-analytics-linktic/dev-bi-proyectosexternos.git`
- Rama: `main`, árbol limpio.
- Único commit: `c38dcdf` "Tablero proyectos externos: login con contraseña y PostgreSQL directo" (John Hernandez, 16-oct-2026).

## Notas

- **Divergencia con la documentación de la entrega**: `documentacion-tecnica.md` describe la versión con Supabase Auth + Google y `supabase-js`; el código actual (15/16-oct-2026) usa `pg` directo y login con contraseña (detalle en `docs/login-y-base-sin-supabase.md`). Los scripts `clear-data` y `unificado-*` siguen usando `supabase-js` con `service_role`.
- Pendientes de seguridad (`PENDIENTES.md`): subir `next` a >= 16.4.0 (vulnerabilidad crítica en 16.2.2), reemplazar `xlsx 0.18.5`, y validar sesión en `/api/ai-analysis*` (hoy públicos).
- `profiles.email` sin índice único; dominios permitidos fijos en `lib/allowed-domains.ts` (`linktic.com`, `3tcapital.co`, `positiva.gov.co`).
- Un módulo con API debe estar en `modulo_fuente_api` **y** en `RPC_UNIFICADO` (`lib/rollup-aggregator.ts`).
