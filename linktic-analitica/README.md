# Linktic — Área de Analítica / BI

Documentación de los proyectos locales en `/Users/johnhernandez/Documents/Linktic/Analitica/`. Hay un archivo por proyecto en esta carpeta. Solo se documentan **nombres** de variables de entorno, nunca sus valores.

## Panorama

El área de BI Analytics construye tableros ejecutivos y APIs de reportes de solo lectura para varios clientes:

- **Balú / ARL Positiva**: tableros de Afiliaciones y Recaudo (back NestJS sobre la réplica PostgreSQL, más una POC con DuckDB/Parquet).
- **Docum / Positiva SGDEA**: 13 tableros de gestión documental sobre BigQuery, más un tablero anterior de facturas recibidas.
- **AgoraCRM**: API de reportes sobre el PostgreSQL de Odoo, más un front recién creado.
- **Fiduprevisora**: microservicio con unos 79 reportes BI del ERP.
- **Patria / Participación Ciudadana**: MVPs de escucha ciudadana con un tablero territorial, todo sobre Supabase.

Patrones comunes:

- **Backs**: NestJS + `pg` sin ORM + Redis opcional + JWT + Docker + Azure Pipelines (despliegue a AWS ECR/ECS). Los más nuevos se montan sobre el harness `arness-back`.
- **Fronts**: Next.js 16 + React 19 + Tailwind 4. Comparten la identidad visual Balú (`tokens_balu`).
- **Remotos**: la mayoría está en Azure DevOps, proyecto "BI Analytics". Algunos están en GitHub (`LinkTic-BI-Analytics`, `bi-analytics-linktic`).

## Proyectos

| Proyecto | Tipo | Cliente / producto | Propósito | Stack | Ruta local |
|---|---|---|---|---|---|
| [dev-back-bi-analytics-balu-dashboard-v1](dev-back-bi-analytics-balu-dashboard-v1.md) | back | Balú / ARL Positiva | API de los tableros Afiliaciones y Recaudo (dataset en memoria sobre la réplica PG `bi.rep_*`) | NestJS 11, Node 22, pg, ioredis, JWT | `Analitica/dev-back-bi-analytics-balu-dashboard-v1` |
| [poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1](poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1.md) | poc | Balú / ARL Positiva | Motor analítico con DuckDB sobre snapshots Parquet en S3, equivalente al back PG | NestJS 11, DuckDB node-api, AWS S3 SDK | `Analitica/poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1` |
| [dev-back-bi-balu-recaudo-v1](dev-back-bi-balu-recaudo-v1.md) | back | Balú / Positiva | Back v2 de reportes (aportantes, afiliados, recaudo diario), con login contra GAL | NestJS 12 ESM, Node 24, pg, ioredis, arness-back | `Analitica/dev-back-bi-balu-recaudo-v1` |
| [dev-bi-front-docum-v1](dev-bi-front-docum-v1.md) | front (full-stack Next) | Docum / Positiva SGDEA | 13 tableros ejecutivos documentales que reemplazan Looker Studio, más el monitor BigQuery `/monit` | Next 16, React 19, Chart.js, Mapbox, TanStack Query, BigQuery | `Analitica/docum/dev-bi-front-docum-v1` |
| [dev-bi-docum](dev-bi-docum.md) | front | Docum / Positiva SGDEA | Copia anterior (GitHub) de dev-bi-front-docum-v1, hasta WI-008 | igual que dev-bi-front-docum-v1 | `Analitica/dev-bi-docum` |
| [dev-bi-back-docum-v1](dev-bi-back-docum-v1.md) | back (vacío) | Docum / Positiva SGDEA | Repo reservado para el back de Docum; hoy solo tiene un README | — | `Analitica/docum/dev-bi-back-docum-v1` |
| [dev-front-bi-analytics-docum-dashboard-v1](dev-front-bi-analytics-docum-dashboard-v1.md) | front (full-stack Next) | Positiva SGDEA (facturación) | Tablero de facturas electrónicas recibidas, con resumen IA de proveedores | Next 16, pg (PostgreSQL vía IAP), OpenAI, SerpAPI | `Analitica/dev-front-bi-analytics-docum-dashboard-v1` |
| [dev-front-bi-agoracrm-v1](dev-front-bi-agoracrm-v1.md) | front | AgoraCRM | Front de los dashboards AgoraCRM (por ahora solo el scaffold) | Next 16, React 19, Tailwind 4, pnpm | `Analitica/dev-front-bi-agoracrm-v1` |
| [dev-back-bi-agoracrm-v1](dev-back-bi-agoracrm-v1.md) | back | AgoraCRM | API de reportes sobre el PostgreSQL de Odoo (crm-leads, sucursales, sale-orders) | NestJS 12 ESM, Node 24, pg, Redis, arness-back | `Analitica/dev-back-bi-agoracrm-v1` |
| [rdt-ms-reporter-bi](rdt-ms-reporter-bi.md) | back | Fiduprevisora (CORE) | Unos 79 reportes BI del ERP sobre vistas materializadas PG | NestJS 11, Node 22, pg, Redis | `Analitica/rdt-ms-reporter-bi` |
| [repot-temp](repot-temp.md) | back (snapshot) | Fiduprevisora (base Balu) | Copia en el commit inicial del reporter (módulo de terceros) | NestJS 11, pg, Redis | `Analitica/repot-temp` |
| [arness-back](arness-back.md) | herramienta interna | Linktic (toda el área) | Harness de agentes IA y CLI `arness` para estandarizar backs NestJS | Node ≥20, sin dependencias | `Analitica/arness-back` |
| [dev-front-patria-milagros](dev-front-patria-milagros.md) | front | Patria / Participación Ciudadana | Tablero territorial con mapa de Colombia, KPIs y narrativas ciudadanas | Next 16, Supabase, Mapbox, Recharts | `Analitica/mvp/dev-front-patria-milagros` |
| [sistema-ciudadano](sistema-ciudadano.md) | full-stack | Patria / Participación Ciudadana | Método de descubrimiento más un producto de captura y revisión de aportes ciudadanos (IA opcional) | Next 15.5, Supabase, Zod, Playwright | `Analitica/mvp/sistema-ciudadano` |
| [sistema-ciudadano-bi](sistema-ciudadano-bi.md) | full-stack | Patria / Participación Ciudadana | Continuación activa: "Sistema de Escucha y Planeación Nacional", con login por token y "Te llamamos" | Next 15.5, Supabase, Zod, Playwright | `Analitica/mvp/sistema-ciudadano-bi` |

`mvp/sistema-ciudadano.git` es un repo bare espejo que se usó para migrar a sistema-ciudadano-bi. Se describe dentro de [sistema-ciudadano](sistema-ciudadano.md).

## Parejas front ↔ back

| Front | Back(s) | Nota |
|---|---|---|
| Front Balú (`front-bi-balu.linktic.com`; sin copia local) | dev-back-bi-analytics-balu-dashboard-v1, dev-back-bi-balu-recaudo-v1 (POC: poc_dbduck_…) | El front lee la variable `NEXT_PUBLIC_AFILIACIONES_API_URL`. Contexto adicional en `~/Documents/README_BALU.md` y `~/Documents/md/Balu/` |
| dev-bi-front-docum-v1 (y su antecesor dev-bi-docum) | dev-bi-back-docum-v1 (vacío) | Hoy la capa de datos BigQuery vive en las API routes de Next |
| dev-front-bi-analytics-docum-dashboard-v1 | — (API routes propias con pg) | Su alcance lo absorbe ahora el tablero `facturas-recibidas` de Docum BI |
| dev-front-bi-agoracrm-v1 | dev-back-bi-agoracrm-v1 | La guía de consumo está en `FRONT.md` del back |
| — (sin front local) | rdt-ms-reporter-bi | Contrato para el front en `FRONT.md` |
| dev-front-patria-milagros | sistema-ciudadano-bi / sistema-ciudadano (Supabase, esquema `participacion`) | Comparten la base. Las migraciones del tablero están desfasadas |

## Otras relaciones

- **arness-back**: es la base de dev-back-bi-agoracrm-v1 y dev-back-bi-balu-recaudo-v1.
- **repot-temp y rdt-ms-reporter-bi**: comparten remoto (`rdt_ms_api_erp_report_generator`). repot-temp es solo la foto del commit inicial.
- **dev-bi-docum (GitHub) y dev-bi-front-docum-v1 (Azure)**: es el mismo producto. El de Azure es el vigente.
- **`/Users/johnhernandez/dev-front-bi-agoracrm-v1`**: no es un repo, solo contiene el README plantilla. La copia válida es la de Analitica.
- **Avisos de seguridad**: rdt-ms-reporter-bi tiene un `.env` real en local. El Dockerfile de dev-front-bi-analytics-docum-dashboard-v1 copia `.env` dentro de la imagen. El `storage/` de la POC DuckDB contiene Parquet con datos de afiliados.
