# Linktic — otros proyectos locales

Documentación de los proyectos locales de Linktic que no forman parte de las otras carpetas de documentación: el backend de reportería (plantilla y POC), Health Apps, el paquete de entrega del Tablero Proyectos Externos y material de referencia (CPE). Estado al 19 de octubre de 2026.

> Seguridad: estos documentos solo incluyen **nombres** de variables de entorno, nunca valores. Varios proyectos tienen archivos `.env*` locales con credenciales y el paquete de proyectos externos tiene datos personales en `01_tablero/datos/`.

## Proyectos

| Nombre | Propósito | Stack | Ruta local |
|---|---|---|---|
| [poc-back-generacion-reportes](poc-back-generacion-reportes.md) | POC del backend del generador dinámico de reportes (solo lectura, cliente → módulo → endpoint), con harness `@linktic/arness-back` | NestJS 12, TypeScript 6, `pg`, Vitest, oxlint, pnpm | `/Users/johnhernandez/Documents/Linktic/poc/poc-back-generacion-reportes` |
| [backend-reporting-template](backend-reporting-template.md) | Plantilla base de reportería; antecedente del POC | NestJS 11, TypeScript 5, `pg`, Jest, ESLint, npm | `/Users/johnhernandez/Documents/backend-reporting-template` |
| [salud-apps (Health Apps)](salud-apps.md) | Monitoreo centralizado de salud, uptime y alertas de aplicaciones | Next.js 16.3, React 19.2, Prisma 7.10, PostgreSQL 18 / Supabase, Vitest, Docker | `/Users/johnhernandez/Documents/Linktic/projectos_alter/salud-apps` |
| [my-app](my-app.md) | Scaffold vacío de `create-next-app` (probable prueba previa a salud-apps) | Next.js 16.3, React 19.2, Tailwind 4 | `/Users/johnhernandez/Documents/Linktic/projectos_alter/my-app` |
| [Tablero Proyectos Externos](tablero-proyectos-externos.md) | Tableros de mesa de ayuda (BALÚ, SGDEA, Fiscalía, UGPP, Equidad, CRM, Core, Rama, Cymetria, TNL) con IA; paquete de entrega 13-oct-2026 | Next.js 16.2, React 19.2, PostgreSQL (`pg`), OpenAI, Recharts | `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/01_tablero` |
| [ETL Data Core](etl-data-core.md) | Ingesta horaria de casos de la API Data Core al tablero y refresco del rollup | Python 3.10+, psycopg2, requests, cron | `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/02_etl_data_core` |
| [supabase-local](supabase-local.md) | Base local del tablero (esquema `tablero_proyectos_externos`) | Supabase CLI, PostgreSQL 17, Docker | `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/supabase-local` |
| [requerimientostablerossoporte](requerimientos-tableros-soporte.md) | Prompts de IA por marca y matriz de ANS para Zendesk | Documentos (`.txt`, `.xlsx`) | `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/requerimientostablerossoporte` |
| [CPE — Anexo técnico SM&E](cpe-sme-anexo-tecnico.md) | Anexo técnico (2022) de Computadores para Educar: módulo de analítica y reportes del SM&E | Documento PDF | `/Users/johnhernandez/Documents/Linktic/CPE` |

## Relaciones entre proyectos

- **Reportería**: `backend-reporting-template` (sep-2026) → `poc-back-generacion-reportes` (oct-2026, mismo patrón, ahora con harness para agentes). El contexto funcional es la API BI de Fiduprevisora (`/Users/johnhernandez/Documents/md/Fidu`) y el catálogo `/Users/johnhernandez/Documents/Catálogo de integración para el generador dinámico de reportes.xlsx` (módulos, tablas, campos, relaciones y queries del CORE fiduciario).
- **Proyectos externos**: el tablero (`01_tablero`) se alimenta del Excel y de la `ETL Data Core`; `supabase-local` da la base local; `requerimientostablerossoporte` trae los requerimientos de prompts y ANS. La documentación general del paquete está en `proyectos_externos/README.md`, `PENDIENTES.md` y `README_MODULO_BALU.md`.
- **Health Apps**: `my-app` es un scaffold sin código; `salud-apps` es el proyecto real (repo `bi-analytics-linktic/dev-healthapps-v2`).

## Repositorios git

| Proyecto | Remoto | Rama | Estado |
|---|---|---|---|
| poc-back-generacion-reportes | sin remoto | `main` | sin commits |
| salud-apps | `bi-analytics-linktic/dev-healthapps-v2` | `fix/db-pool-vercel` | limpio |
| tablero-proyectos-externos (código) | `bi-analytics-linktic/dev-bi-proyectosexternos` | `main` | 1 commit, limpio |
| Resto | no son repositorios propios | — | — |
