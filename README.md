# Documentación general de proyectos

Inventario y documentación de todos los proyectos ubicados en `~/Documents`, generado el 2026-10-19.
Cada proyecto tiene su propio archivo `.md` dentro de la carpeta de su área, y cada área tiene un `README.md` con su índice.

> Las variables de entorno se documentan solo por **nombre**; ningún archivo contiene valores secretos.

## Áreas

| Carpeta | Contenido | Proyectos |
|---|---|---|
| [linktic-fiduprevisora/](linktic-fiduprevisora/README.md) | Ecosistema de microservicios ERP de FiduPrevisora (Laravel) y sus copias de prueba | 10 |
| [linktic-analitica/](linktic-analitica/README.md) | Área BI/Analítica: Balú, Docum, AgoraCRM, Patria, Fiduprevisora BI | 15 |
| [linktic-otros/](linktic-otros/README.md) | Generador de reportes, Health Apps, tableros de proyectos externos, CPE | 9 |
| [ale/](ale/README.md) | Proyectos personales: Mercadito, Asistencia, Linux Quest, POCs y ejercicios | 10 |
| [recursos/](recursos/README.md) | Línea gráfica Patria, notas `md/`, Obsidian y archivos de referencia sueltos | 4 docs |

---

## Linktic · FiduPrevisora (ERP)

| Proyecto | Propósito | Stack |
|---|---|---|
| [rdt_ms_api_erp_core](linktic-fiduprevisora/rdt_ms_api_erp_core.md) | Orquestador Docker/Nginx del ERP (~33 micros) y dueño del esquema (~1.246 migraciones) | Laravel 11, PHP 8.2, PostgreSQL 15, Redis, MinIO, Typesense |
| [rdt_ms_api_erp_core_package](linktic-fiduprevisora/rdt_ms_api_erp_core_package.md) | Paquete Composer `fiduprevisora/core`: modelos, cliente HTTP entre micros, middlewares, auditoría | Paquete Laravel, Pest |
| [rdt_ms_api_erp_users](linktic-fiduprevisora/rdt_ms_api_erp_users.md) | Identidad, usuarios, roles, permisos, 2FA, tokens Sanctum | Laravel 12, Sanctum, Scout + Typesense |
| [rdt_ms_api_erp_security](linktic-fiduprevisora/rdt_ms_api_erp_security.md) | Login local/LDAP/Azure AD, ciclo de vida de tokens, dashboards de seguridad | Laravel 12, Sanctum, OTel |
| [rdt_ms_api_erp_thirdparties](linktic-fiduprevisora/rdt_ms_api_erp_thirdparties.md) | Terceros, KYC/FATCA, importación masiva, SARLAFT/RUES/Inspektor | Laravel 11, Reverb, S3 |
| [rdt_ms_api_erp_reporter](linktic-fiduprevisora/rdt_ms_api_erp_reporter.md) | Reportes Excel asíncronos (25 definiciones) a S3 con aviso por WebSocket | Laravel 11, PhpSpreadsheet |
| [rdt_ms_api_erp_broadcasting](linktic-fiduprevisora/rdt_ms_api_erp_broadcasting.md) | Servidor WebSocket para notificaciones en tiempo real | Laravel 11, Reverb |
| [Test/rdt_ms_api_erp_core](linktic-fiduprevisora/test-rdt_ms_api_erp_core.md) | Duplicado del core (mismo commit, `develop`) | — |
| [Test/rdt_ms_api_erp_reporter](linktic-fiduprevisora/test-rdt_ms_api_erp_reporter.md) | Duplicado del reporter (`chore/report-library`) | — |
| [testfidu/rdt_ms_api_erp_core](linktic-fiduprevisora/testfidu-rdt_ms_api_erp_core.md) | Clon SSH de prueba en `master` (congelado 2025-02-20) | — |

## Linktic · Analítica / BI

| Proyecto | Cliente | Propósito | Stack |
|---|---|---|---|
| [dev-back-bi-analytics-balu-dashboard-v1](linktic-analitica/dev-back-bi-analytics-balu-dashboard-v1.md) | Balú / ARL Positiva | API de tableros Afiliaciones y Recaudo | NestJS 11, pg, Redis |
| [poc_dbduck_…balu-dashboard-v1](linktic-analitica/poc_dbduck_dev-back-bi-analytics-balu-dashboard-v1.md) | Balú | POC: agregación con DuckDB sobre Parquet en S3 | NestJS 11, DuckDB |
| [dev-back-bi-balu-recaudo-v1](linktic-analitica/dev-back-bi-balu-recaudo-v1.md) | Balú / Positiva | Back v2 de aportantes, afiliados y recaudo | NestJS 12, arness-back |
| [dev-bi-front-docum-v1](linktic-analitica/dev-bi-front-docum-v1.md) | Docum / Positiva SGDEA | 13 tableros documentales y monitor BigQuery | Next 16, BigQuery, Chart.js |
| [dev-bi-docum](linktic-analitica/dev-bi-docum.md) | Docum | Copia anterior de dev-bi-front-docum-v1 | Next 16 |
| [dev-bi-back-docum-v1](linktic-analitica/dev-bi-back-docum-v1.md) | Docum | Repo reservado (solo README) | — |
| [dev-front-bi-analytics-docum-dashboard-v1](linktic-analitica/dev-front-bi-analytics-docum-dashboard-v1.md) | Positiva SGDEA | Tablero de facturas electrónicas con resumen IA | Next 16, pg, OpenAI |
| [dev-front-bi-agoracrm-v1](linktic-analitica/dev-front-bi-agoracrm-v1.md) | AgoraCRM | Front de dashboards (scaffold) | Next 16 |
| [dev-back-bi-agoracrm-v1](linktic-analitica/dev-back-bi-agoracrm-v1.md) | AgoraCRM | API de reportes sobre PostgreSQL de Odoo | NestJS 12, arness-back |
| [rdt-ms-reporter-bi](linktic-analitica/rdt-ms-reporter-bi.md) | Fiduprevisora | ~79 reportes BI sobre vistas materializadas | NestJS 11, pg, Redis |
| [repot-temp](linktic-analitica/repot-temp.md) | Fiduprevisora | Snapshot inicial del reporter BI | NestJS 11 |
| [arness-back](linktic-analitica/arness-back.md) | Linktic (interno) | Agentes IA + CLI para estandarizar backs NestJS | Node ≥20 |
| [dev-front-patria-milagros](linktic-analitica/dev-front-patria-milagros.md) | Patria | Tablero territorial con mapa, KPIs y narrativas | Next 16, Supabase, Mapbox |
| [sistema-ciudadano](linktic-analitica/sistema-ciudadano.md) | Patria | Captura y revisión de aportes ciudadanos | Next 15, Supabase |
| [sistema-ciudadano-bi](linktic-analitica/sistema-ciudadano-bi.md) | Patria | Continuación activa: Sistema de Escucha y Planeación Nacional | Next 15, Supabase |

## Linktic · Otros proyectos

| Proyecto | Propósito | Stack |
|---|---|---|
| [poc-back-generacion-reportes](linktic-otros/poc-back-generacion-reportes.md) | POC del back del generador dinámico de reportes (solo infraestructura por ahora) | NestJS 12, pg, arness-back |
| [backend-reporting-template](linktic-otros/backend-reporting-template.md) | Plantilla base de backs de reportes, antecesora de la POC | NestJS 11, pg, Jest |
| [salud-apps](linktic-otros/salud-apps.md) | Monitoreo de salud, uptime y alertas de aplicaciones | Next 16, Prisma 7, PostgreSQL/Supabase |
| [my-app](linktic-otros/my-app.md) | Starter vacío de `create-next-app` | Next 16 |
| [tablero-proyectos-externos](linktic-otros/tablero-proyectos-externos.md) | Tableros de mesa de ayuda con análisis IA (BALÚ, SGDEA, UGPP…) | Next 16, pg, OpenAI |
| [etl-data-core](linktic-otros/etl-data-core.md) | ETL horario de casos Data Core API a la BD del tablero | Python, psycopg2, cron |
| [supabase-local](linktic-otros/supabase-local.md) | Esquema local de la BD del tablero | Supabase CLI, PostgreSQL 17 |
| [requerimientos-tableros-soporte](linktic-otros/requerimientos-tableros-soporte.md) | Prompts IA por marca y matriz ANS Zendesk | Documentos |
| [cpe-sme-anexo-tecnico](linktic-otros/cpe-sme-anexo-tecnico.md) | Anexo técnico (PDF) del módulo de analítica SM&E de Computadores para Educar | PDF |

## Ale (proyectos personales)

| Proyecto | Propósito | Stack |
|---|---|---|
| [asistencia](ale/asistencia.md) | Asistencia a talleres por QR, calificaciones y panel admin | Next 16, Prisma 7, PostgreSQL 17 |
| [list-shopping (Mercadito)](ale/list-shopping.md) | App móvil offline-first de listas de compras con asistente IA | Expo 57, React Native, SQLite, OpenAI |
| [landing-list-shopping](ale/landing-list-shopping.md) | Landing de descarga de Mercadito | Astro 7, Tailwind 4 |
| [mercadito-prototipo-branding](ale/mercadito-prototipo-branding.md) | Prototipo de pantallas y guía de marca (3 copias idénticas) | HTML/CSS/JS |
| [knowledge-mercadito](ale/knowledge-mercadito.md) | Base de conocimiento: negocio, roadmap, work items, ADRs | Markdown, Obsidian, Kaddo |
| [interfaz (Linux Quest)](ale/interfaz.md) | Juego web para aprender la terminal Linux | HTML/CSS/JS |
| [prueba-velocidadquerys](ale/prueba-velocidadquerys.md) | POC: medir una query SQL pesada sobre terceros | NestJS 12, pg |
| [conexion-gcp-bidquery](ale/conexion-gcp-bidquery.md) | POC: conexión NestJS a BigQuery | NestJS 12, BigQuery |
| [jugueteria](ale/jugueteria.md) | Ejercicio: peso de envío de pedidos | Python 3 |
| [draw-io](ale/draw-io.md) | Skill de diagramas draw.io + diagramas AWS de BALU | draw.io, Bash, Python |

## Recursos

- [linea-grafica-patria.md](recursos/linea-grafica-patria.md): sistema de diseño "Patria Milagro" (colores, tipografía, tokens, componentes).
- [md-notas.md](recursos/md-notas.md): notas de Balú y Fidu en `~/Documents/md`.
- [documentacion-obsidian.md](recursos/documentacion-obsidian.md): la bóveda está vacía.
- [archivos-referencia.md](recursos/archivos-referencia.md): PDFs, Excels, exportaciones y zips sueltos.

---

## Relaciones entre proyectos

- **arness-back** es la base de `dev-back-bi-agoracrm-v1`, `dev-back-bi-balu-recaudo-v1` y `poc-back-generacion-reportes`.
- **backend-reporting-template** es el antecesor de **poc-back-generacion-reportes**.
- **Patria**: `sistema-ciudadano`/`sistema-ciudadano-bi` alimentan la base Supabase que lee `dev-front-patria-milagros`; la identidad visual está en `recursos/linea-grafica-patria.md`.
- **Mercadito**: app (`list-shopping`) + landing + prototipo/branding + base de conocimiento.
- **Tablero proyectos externos**: front `01_tablero` + `etl-data-core` + `supabase-local` + `requerimientos-tableros-soporte`.
- `~/dev-front-bi-agoracrm-v1` (en el home) **no** es el repo real; la copia válida está en `Linktic/Analitica`.

## Hallazgos a revisar

**Seguridad**
- `list-shopping`: la clave de OpenAI viaja en el bundle (`EXPO_PUBLIC_IA_APIKEY`); requiere backend/proxy antes de publicar.
- `dev-front-bi-analytics-docum-dashboard-v1`: el Dockerfile copia `.env` dentro de la imagen.
- `rdt_ms_api_erp_broadcasting`: la ruta pública `/s3/buckets` lista buckets sin autenticación.
- `tablero-proyectos-externos`: `next` 16.2.2 (vulnerabilidad crítica), `xlsx` 0.18.5 (alta) y `/api/ai-analysis*` sin validar sesión.
- Datos personales en local: Parquet de afiliados en la POC DuckDB (`storage/`) y la carpeta `datos/` del tablero de proyectos externos.
- Hay `.env` con credenciales reales en `rdt-ms-reporter-bi`, `salud-apps`, el tablero y el ETL.

**Estado / documentación**
- La rama por defecto del core ERP (`master`) está obsoleta; se trabaja en `develop`.
- El reporter ERP está en `chore/report-library` y su `ARCHITECTURE.md`/`README.md` están desactualizados.
- La base de conocimiento de Mercadito describe la app como "100% offline", pero el código ya usa OpenAI.
- La documentación de entrega del tablero externo describe Supabase Auth con Google; el código usa PostgreSQL directo con email y contraseña.
- La Data Core API dejó de responder el 2026-10-16 (error de DNS).
- Varios proyectos tienen trabajo sin commitear (`list-shopping`, `landing-list-shopping`, la POC DuckDB, `dev-bi-docum`) o no son repos propios (solo están dentro del repo git del home).
