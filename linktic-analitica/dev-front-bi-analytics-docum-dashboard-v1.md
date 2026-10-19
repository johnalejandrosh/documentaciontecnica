# dev-front-bi-analytics-docum-dashboard-v1 — Dashboard de Facturación Electrónica Positiva (Docum)

- **Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/dev-front-bi-analytics-docum-dashboard-v1`
- **Tipo:** Front (Next.js full-stack: UI + API routes que leen PostgreSQL)
- **Cliente / producto:** Positiva Compañía de Seguros · SGDEA "Docum" — facturación electrónica recibida

## Descripción / propósito

Dashboard gerencial de la **facturación electrónica RECIBIDA** por Positiva (los proveedores facturan a Positiva; `proveedor` = NIT del emisor). Desarrollado por BI Analytics con la identidad gráfica **Balú** (la del dashboard Digiturno de Balu). Es el reemplazo directo del antiguo `tablero-facturacion-positiva` (Next 16.0 + glassmorphism), replicando su funcionalidad y corrigiendo bugs.

Primera entrega: 2026-08-06.

## Stack y versiones

| Pieza | Versión |
|---|---|
| Next.js (App Router, Turbopack, salida standalone) | 16.3.0 |
| React | 19.2.8 |
| TypeScript (strict) | 5 |
| Tailwind CSS | v4 (CSS-first, sin tailwind.config) |
| chart.js + react-chartjs-2 | 4.5 / 5.3 |
| pg (PostgreSQL, solo lectura, solo servidor) | 8.22 |
| Gestor | pnpm 11.18.0 |

Sin librería de UI, iconos ni fechas: primitivas propias en `components/ui/`, iconos SVG inline, fechas `Intl` (TZ America/Bogota).

## Arquitectura y estructura de carpetas

```
src/app                 page.tsx (dashboard), login, api/*
src/components          filters, kpis, diaria, estados, dependencias, proveedores, facturas, anual, dashboard, layout, auth, ui
src/lib/db/pool.ts      pool pg
src/lib/data            source.ts (conmutador real/demo), real.ts, demo/ (datos deterministas), codec.ts
src/lib/auth            jwt.ts, session.ts, cliente.ts
src/lib/metrics, domain cálculos y modelo de dominio
src/providers, hooks, types
public/graphic_identity/tokens_balu
Dockerfile              build multi-etapa node:22-alpine, standalone, healthcheck /health
```

Documentación interna: `CLAUDE.md` (arquitectura de datos, esquema de BD, fuente demo, decisiones, gotchas, roadmap) y `AGENTS.md`.

## Funcionalidades / pantallas / endpoints

**Pantallas**
- `/login` — acceso con token compartido (fail-closed si no está configurado).
- `/` — dashboard: KPIs del periodo con variación vs periodo anterior, comparación diaria, estados y dependencias, ranking de proveedores con detalle e IA, listado de facturas con filtros avanzados, vista anual comparativa. Conmutador Datos reales / Demo en el header. Tema claro/oscuro.

**API**
- `POST /api/auth/login`, `POST /api/auth/logout`
- `GET /api/facturacion/dataset`
- `GET /api/facturacion/anual`
- `GET /api/facturacion/proveedor/[nit]`
- `GET /api/facturacion/proveedor-info` — búsqueda del NIT en Google (SerpAPI) + resumen con OpenAI (gpt-4o-mini); opcional
- `GET /api/health` (healthcheck de Docker)

## Fuentes de datos e integraciones

- **PostgreSQL** de Positiva en GCP, BD `ms_facturacion_core`, tabla `factura_recibida` (solo lectura). En desarrollo se accede por **túnel IAP** (`gcloud compute ssh … --tunnel-through-iap -L 5432:…`), ver `CLAUDE.md`.
- **Datos demo** deterministas (sin BD).
- **SerpAPI** y **OpenAI** (opcionales) para la ficha del proveedor.

## Cómo ejecutarlo localmente

```bash
pnpm install
cp .env.example .env    # completar credenciales
# levantar túnel IAP para datos reales, o usar modo Demo
pnpm dev                # http://localhost:3000
```
Scripts: `pnpm dev`, `pnpm build` (no corre ESLint en Next 16), `pnpm start`, `pnpm lint`.

Docker: `docker build -t docum-dashboard .` y `docker run -p 3000:3000 docum-dashboard` (el Dockerfile copia `.env` a la imagen).

## Variables de entorno (solo nombres)

`DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `TOKEN_ACCESS`, `JWT_SECRET` (opcional), `SERPAPI_KEY` (opcional), `OPENAI_API_KEY` (opcional).

## Pruebas

No hay suite de pruebas ni script `test`; solo `pnpm lint`.

## Estado git

- **Remoto:** Azure DevOps `Linktic/004 - POSITIVA SGDEA/_git/dev-next-dashboard` (proyecto Positiva SGDEA, **no** en BI Analytics; el nombre del repo remoto es `dev-next-dashboard`).
- **Rama actual:** `feature/add-token-login` (otras: `master`, `feature/andres`)
- **Commits (4, 2026-08-06):** `99587c2` adjust · `22211e0` adlogin · `9d1f7b5` add Dockerfile (John Hernandez) · `4a6c6db` first commit (Andrés Ramírez)

## Notas

- **Predecesor de Docum BI:** su alcance (facturas recibidas) quedó absorbido por el tablero `facturas-recibidas` de [`dev-bi-front-docum-v1`](./dev-bi-front-docum-v1.md), que lee la vista BigQuery `vw_reporte_datastudio_facturacion_factura_recibida` en lugar de Postgres directo.
- Sin back separado: el acceso a datos está en las API routes del propio Next.js.
- Comparte identidad gráfica Balú con `dev-front-bi-analytics-balu-dashboard-v1`.
- Cuidado: el Dockerfile empaqueta `.env` dentro de la imagen.
