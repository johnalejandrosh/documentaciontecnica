# dev-bi-docum — Docum BI (copia en GitHub, rama de migración a BigQuery)

- **Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/dev-bi-docum`
- **Tipo:** Front (Next.js full-stack)
- **Cliente / producto:** Positiva Compañía de Seguros · SGDEA "Docum"
- **Paquete:** `front-bi-analytics-docum` v0.1.0

## Descripción / propósito

Mismo producto que [`dev-bi-front-docum-v1`](./dev-bi-front-docum-v1.md): los 13 tableros ejecutivos de seguimiento de flujos documentales de Positiva (facturación electrónica, PQRD, entes de control, SMART Supervisión M1–M3, tutelas, medicina laboral, correspondencia), que reemplazan los tableros de Looker Studio.

Este clon apunta al repositorio de **GitHub** (`LinkTic-BI-Analytics/dev-bi-docum`) y quedó en el punto de la migración mock → BigQuery (WI-000 a WI-008). Es un **estado anterior** de `dev-bi-front-docum-v1`: no incluye el monitor `/monit` ni el refresco controlado de datasets.

## Stack y versiones

Idéntico a `dev-bi-front-docum-v1` (mismo `package.json`): Next.js 16.3.6, React 19.2.8, TypeScript 5, Tailwind v4, Chart.js 4.5 (+ sankey/treemap/datalabels), Mapbox GL 3.31, TanStack Query 5, motion 13, jose 6, @google-cloud/bigquery 9.1, vitest 5, pnpm 12.4.1.

## Arquitectura y estructura de carpetas

Misma arquitectura declarativa: `src/dashboards/specs` (13 tableros) → motor `src/server/data/engine` → `DataProvider` (mock | BigQuery). Carpetas: `src/app`, `src/dashboards`, `src/components` (sin `monit`), `src/server` (auth, data), `src/lib`, `scripts/mock`, `scripts/geo`, `public/data/geo`, `docs`, `top_secret/` (ignorado).

## Funcionalidades / pantallas / endpoints

- Pantallas: `/login`, `/` (Centro de mando documental), `/tableros/[slug]`.
- API: `/api/catalogo`, `/api/tableros/[slug]`, `/api/tableros/[slug]/detalle`, `/api/salud/bigquery`.
- (No tiene `/monit` ni `/api/monit/*`.)

## Fuentes de datos e integraciones

- BigQuery (vistas `oro_tableros.vw_reporte_datastudio_*` de Positiva SGDEA, solo lectura, ADC) o mock anonimizado, según `DATA_SOURCE`.
- Mapbox para el mapa de Colombia.

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env.local
pnpm dev        # http://localhost:3000
```
Scripts: `dev`, `build`, `start`, `lint`, `test` (vitest), más `scripts/mock/build-profiles.mjs` y `scripts/geo/*.mjs`.

## Variables de entorno (solo nombres)

`ACCESS_TOKEN`, `SESSION_SECRET`, `SESSION_TTL_HOURS`, `NEXT_PUBLIC_MAPBOX_TOKEN`, `NEXT_PUBLIC_MAPBOX_STYLE`, `NEXT_PUBLIC_MAPBOX_STYLE_DARK`, `DATA_SOURCE`, `GCP_PROJECT_ID`, `BQ_DATASET`, `BQ_LOCATION`, `BQ_MAX_BYTES_BILLED`, `BQ_CACHE_TTL_MIN`, `BQ_CACHE_MAX_STALE_MIN`, `CSV_MAX_ROWS`.

## Pruebas

Vitest: métricas congeladas contra `baseline.json`, layout, semántica, formatos, motor y provider BigQuery (incluye pruebas de integración/QA contra BigQuery real).

## Estado git

- **Remoto:** GitHub `LinkTic-BI-Analytics/dev-bi-docum`
- **Rama actual:** `feature/conexionbigquery` (también `main`)
- **Cambios locales sin commit:** `M src/server/data/bigquery/README.md`, `?? docs/bigquery-consultas.md`
- **Últimos commits:** `37e6b05` WI-008 QA contra BigQuery real, rendimiento y costo · `b32efe7` WI-007 controles de seguridad y datos personales · `e42bfeb` WI-006 frescura del dato · `9412d48` WI-005 BigQueryProvider con snapshot, TTL y single-flight · `1f2b391` WI-004 · `76780e9` WI-003 contrato de datos · `a4d1fef` WI-002 conexión Next.js ↔ BigQuery · `1cd5fdd` WI-001 selector `DATA_SOURCE` · `8b713e5` WI-000 plan de migración · `8f6882e` first commit (último: 2026-10-02).

## Notas

- **Duplicado / antecesor** de `docum/dev-bi-front-docum-v1` (Azure DevOps), que tiene estos mismos 10 commits + 6 posteriores. El trabajo nuevo se está haciendo en el repo de Azure; los cambios locales sin commit aquí (`docs/bigquery-consultas.md`) ya están versionados allá como "docs: inventario de consultas a BigQuery".
- Sin back propio; ver notas de `dev-bi-front-docum-v1`.
