# dev-bi-front-docum-v1 — Docum BI (13 tableros de flujos documentales, Positiva)

- **Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/docum/dev-bi-front-docum-v1`
- **Tipo:** Front (Next.js full-stack: UI + API routes que leen BigQuery)
- **Cliente / producto:** Positiva Compañía de Seguros · SGDEA "Docum"
- **Paquete:** `front-bi-analytics-docum` v0.1.0

## Descripción / propósito

Front-end de los **13 tableros ejecutivos** de seguimiento de flujos documentales de SGDEA/Docum para Positiva. Reemplaza los tableros de Looker Studio (Data Studio) con una UI moderna, coherente y responsiva. Módulos cubiertos: facturación electrónica (recibidas y emitidas), PQRD, entes de control (y su eficiencia operativa), SMART Supervisión (momentos 1, 2 y 3, Superfinanciera), tutelas, medicina laboral (entradas y salidas) y correspondencia (entradas y salidas).

Es la versión **activa y más avanzada** del producto: contiene toda la historia de `dev-bi-docum` (GitHub) más el monitor de consultas BigQuery (`/monit`) y el refresco controlado de datasets.

## Stack y versiones

| Pieza | Versión |
|---|---|
| Next.js (App Router, Turbopack, `src/proxy.ts` en lugar de middleware) | 16.3.6 |
| React | 19.2.8 |
| TypeScript | 5 |
| Tailwind CSS | v4 (CSS-first, tokens en `src/app/globals.css`) |
| Chart.js + react-chartjs-2 (+ sankey, treemap, datalabels) | 4.5 / 5.3 |
| Mapbox GL | 3.31 |
| TanStack Query | 5 |
| motion | 13 |
| jose (JWT de sesión HS256) | 6 |
| @google-cloud/bigquery | 9.1 |
| html-to-image, date-fns | 1.11 / 4 |
| vitest | 5 |
| Gestor | pnpm 12.4.1 · Node ≥ 20 (probado con 24) |

## Arquitectura y estructura de carpetas

Los tableros se definen de forma **declarativa** (`src/dashboards/specs/*.ts`); el servidor los ejecuta con un motor de agregación (`src/server/data/engine`) sobre un `DataProvider` intercambiable (mock o BigQuery, según `DATA_SOURCE`).

```
src/app            rutas: login, catálogo (home), tableros/[slug], monit, API
src/dashboards     contrato declarativo, layout (filas que suman 12) y specs de los 13 tableros; baseline.json de métricas
src/components     shell, home, dashboard, widgets (HTML, Chart.js, Mapbox), monit, brand, ui
src/server/auth    sesión JWT + DAL
src/server/data    provider, engine, datasets, mock, bigquery (client, query, sql, cache, load, gcp-jobs, query-log)
src/lib            fechas (hora Bogotá), formatos es-CO, filtros, gráficas, diccionario geográfico (DIVIPOLA)
scripts/mock       generación de perfiles anonimizados para datos mock
scripts/geo        simplificación de GeoJSON de Colombia y catálogo DIVIPOLA
public/data/geo    GeoJSON (originales + simplificados)
public/graphic_identity/tokens_balu  identidad gráfica Balú
docs               sistema de diseño UI, contrato de datos BigQuery, plan de migración, QA, seguridad, inventario de consultas
top_secret/        (ignorado por git) CSV reales, SQL, pantallazos, runbook — NO versionar
```

Documentación interna clave: `AGENTS.md` (guía completa de negocio y técnica), `CLAUDE.md`, `docs/ui-design-system.md`, `src/server/data/bigquery/README.md`.

## Funcionalidades / pantallas / endpoints

**Pantallas**
- `/login` — acceso con token único.
- `/` — Home "Centro de mando documental": módulos con color de identidad, pulso del mes y salud de cada tablero.
- `/tableros/[slug]` — cada uno de los 13 tableros: `facturas-recibidas`, `facturas-emitidas`, `pqrd`, `entes-control`, `entes-control-eficiencia`, `smart-momento-1/2/3`, `tutelas`, `medicina-laboral-entradas`, `medicina-laboral-salidas`, `correspondencia-entradas`, `correspondencia-salidas`.
- `/monit` — monitor de consultas a BigQuery (costos, jobs GCP) y refresco manual de datasets.

Características: visual protagonista por tablero (mapa de Colombia con insights, series, matrices), filtros cruzados sincronizados con la URL, KPIs con variación vs periodo anterior, tema claro/oscuro, exportación CSV/PNG, semáforo de riesgo y categorías SLA.

**API (todas exigen sesión; 401 sin ella)**
- `GET /api/catalogo`
- `GET /api/tableros/[slug]` y `GET /api/tableros/[slug]/detalle`
- `GET /api/salud/bigquery` — verifica permisos y costo
- `GET /api/monit`, `/api/monit/gcp`, `POST /api/monit/refrescar`

## Fuentes de datos e integraciones

- **BigQuery** (solo lectura): vistas `oro_tableros.vw_reporte_datastudio_*` del proyecto GCP de Positiva SGDEA (región `us-east1`). Credenciales por ADC (`gcloud auth application-default login`). Snapshot en memoria con TTL, single-flight, caché opcional en disco y tope de bytes facturados. Al arrancar precarga 12 datasets (~1 GB facturado).
- **Mock** (por defecto): datos sintéticos anonimizados con semilla fija (fuente de CI y demos).
- **Mapbox** (estilos claro/oscuro del cliente).

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env.local   # completar valores
pnpm dev                     # http://localhost:3000
```

| Script | Uso |
|---|---|
| `pnpm dev` / `pnpm build` / `pnpm start` | desarrollo / build / producción |
| `pnpm lint` | ESLint |
| `pnpm test` | Vitest |
| `node scripts/mock/build-profiles.mjs` | regenera perfiles mock desde `top_secret/db` |
| `node scripts/geo/build-geo.mjs` | genera geografía simplificada y DIVIPOLA |
| `node scripts/geo/check-dictionary.mjs` | mide cruce del diccionario geográfico |

Antes de cerrar un cambio: `npx tsc --noEmit`, `pnpm lint`, `pnpm test`. No hay Dockerfile en el repo.

## Variables de entorno (solo nombres)

`ACCESS_TOKEN`, `SESSION_SECRET`, `SESSION_TTL_HOURS`, `NEXT_PUBLIC_MAPBOX_TOKEN`, `NEXT_PUBLIC_MAPBOX_STYLE`, `NEXT_PUBLIC_MAPBOX_STYLE_DARK`, `DATA_SOURCE` (`mock` | `bigquery`), `GCP_PROJECT_ID`, `BQ_DATASET`, `BQ_LOCATION`, `BQ_MAX_BYTES_BILLED`, `BQ_CACHE_TTL_MIN`, `BQ_CACHE_MAX_STALE_MIN`, `BQ_DISK_CACHE`, `CSV_MAX_ROWS`.

## Pruebas

Vitest (19 archivos de prueba): métricas de los 13 tableros congeladas contra `src/dashboards/baseline.json`, validación de layout (`validateLayout`), registro semántico, formatos, motor de agregación, provider, caché/SQL/jobs de BigQuery. Hay pruebas de integración contra BigQuery real (`*.integration.test.ts`, QA de rendimiento y costo).

## Estado git

- **Remoto:** Azure DevOps `Linktic/BI Analytics/_git/dev-bi-front-docum-v1`
- **Rama actual:** `feature/bq-refresco-controlado` (otras: `main`, `feature/conexionbigquery`, `feature/monit`, `docs/bigquery-consultas`)
- **Árbol:** limpio
- **Últimos commits (2026-10-19):**
  - `1d471ae` feat: refresco manual de datasets desde /monit
  - `b92e79b` feat: el topbar ya no muestra "Datos de hace N min"
  - `2af19ab` feat: gestor de cargas de BigQuery sin reintentos en cascada
  - `6525184` feat: monitor de consultas a BigQuery en /monit
  - `fde1321` Merged PR 238155: docs: inventario de consultas a BigQuery
  - `37e6b05`…`8f6882e` WI-000 a WI-008 (migración mock → BigQuery), compartidos con `dev-bi-docum`

## Notas

- **Relación con `dev-bi-docum`:** mismo código base y mismo `package.json`; `dev-bi-docum` (GitHub) contiene los 10 primeros commits (hasta WI-008) y este repo (Azure DevOps) es su continuación con 6 commits más (monitor `/monit`, refresco controlado, inventario de consultas). Tratar este como el repositorio vigente.
- **Par back:** `docum/dev-bi-back-docum-v1` existe pero está vacío (solo README plantilla de Azure); hoy el "back" vive dentro de este Next.js (API routes + BigQuery).
- **Predecesor:** `dev-front-bi-analytics-docum-dashboard-v1` (dashboard de facturación recibida sobre PostgreSQL); su funcionalidad quedó cubierta por el tablero `facturas-recibidas`.
- Usa la identidad gráfica Balú (`public/graphic_identity/tokens_balu`), compartida con los dashboards Balu.
- `top_secret/` y `.env*` (salvo `.env.example`) están ignorados por git.
