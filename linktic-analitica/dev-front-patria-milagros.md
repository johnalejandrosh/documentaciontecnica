# dev-front-patria-milagros

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/mvp/dev-front-patria-milagros`
> Tipo: front (tablero BI) · Cliente/producto: Patria / "Defensores de la Patria" (Participación Ciudadana Colombia)

## Descripción / propósito

Tablero territorial ("Sistema de Planeación Nacional") de la participación ciudadana en Colombia. Un mapa Mapbox muestra el país por departamentos; con doble clic en un departamento se abren sus municipios. El panel lateral resume aportes, necesidades, alertas, temas, evolución y estado de atención del territorio seleccionado, más las narrativas ciudadanas y su "generalidad", que se obtiene contando y no generando con un modelo.

Es la capa de **BI de lectura** sobre la base de datos que alimenta el producto `sistema-ciudadano` (captura de aportes).

## Stack y versiones

| Pieza | Versión |
|---|---|
| Next.js (App Router) | 16.3.5 |
| React / React DOM | 19.2.8 |
| TypeScript | ^5 |
| Tailwind CSS | ^4 (`@tailwindcss/postcss`) |
| Supabase JS | ^2.116.0 (solo servidor, `server-only`) |
| Mapbox GL | ^3.30.0 |
| Recharts | ^3.10.1 |
| motion, lucide-react | ^13.3.0, ^1.46.0 |
| Gestor de paquetes | pnpm 12.4.1 |

`AGENTS.md` advierte que esta versión de Next trae cambios incompatibles; por ejemplo, el middleware se llama `src/proxy.ts`.

## Arquitectura y estructura

```
src/
  proxy.ts                 # puerta de sesión: sin cookie válida redirige a /acceso
  app/(tablero)/           # page, loading, error: el tablero principal
  app/acceso/              # login con token único (server action)
  components/mapa/         # MapaColombia (Mapbox)
  components/tablero/      # Tablero, TarjetasKpi, Graficas, Narrativas, ControlesMapa, Encabezado, CifraAnimada
  lib/datos/               # tablero.ts (lectura paginada de Supabase), agregar.ts (reglas R1/R2),
                           # narrativas.ts, catalogos.ts, tipos.ts
  lib/auth/sesion.ts       # cookie de sesión firmada
  lib/supabase/servidor.ts # cliente Supabase con llave secreta
  lib/geo/cajas.json       # bounding boxes por departamento
baas/                      # copia del esquema SQL (migración 20260915215046) + BASE-DE-DATOS.md
scripts/preparar-geo.mjs   # genera geojson livianos y cajas
public/data/               # geojson DANE (departamentos/municipios) originales y livianos
public/linea-grafica-patria/ # línea gráfica: tokens, logos, capturas, documentación
```

## Funcionalidades / pantallas

- **`/acceso`**: ingreso con un token único (`ACCESS_TOKEN`), que deja una cookie de sesión firmada.
- **`/`**: el tablero, con mapa coroplético por departamento y municipio, tarjetas KPI, gráficas (temas, evolución, estado de atención), alertas y narrativas ciudadanas.
- Reglas de conteo (en `agregar.ts`): un aporte en varios municipios cuenta una vez por territorio, y los aportes sin ubicación confirmada suman al total nacional pero no al mapa. Sin filtros, los totales se contrastan con el RPC `participacion.indicadores`.

## Fuentes de datos e integraciones

- **Supabase (Postgres)**: esquema `participacion` (aporte, sintesis, ubicacion, expediente, alerta, actuacion, territorio, etc.). Se lee desde el servidor con la llave secreta porque el esquema no da permisos a `anon`. La lectura va paginada de a 1000 filas (tope de PostgREST).
- **Mapbox**: mapa base y estilo.
- **Cartografía DANE / DIVIPOLA**: el cruce con la base va por código (`DPTO`/`MPIOS` = `territorio.codigo`).

## Cómo ejecutarlo localmente

```bash
cp .env.example .env.local   # completar valores
pnpm install
pnpm dev                     # next dev
pnpm build && pnpm start
pnpm lint
node scripts/preparar-geo.mjs   # regenerar cartografía liviana
```

## Variables de entorno (solo nombres)

`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`, `NEXT_PUBLIC_MAPBOX_TOKEN`, `NEXT_PUBLIC_MAPBOX_STYLE`, `ACCESS_TOKEN`, `SESSION_SECRET`

## Pruebas

No hay suite de pruebas. Solo hay `lint` (ESLint 9 + eslint-config-next).

## Estado git

- Remoto: `https://github.com/LinkTic-BI-Analytics/dev-front-patria-milagros.git`
- Rama: `main` (árbol limpio)
- Commits (3): `af25655 Agrega narrativas ciudadanas y su generalidad bajo el mapa`, `5f4faea first commit`, `5f05b61 Initial commit from Create Next App`
- Último commit: 2026-09-16, por Andrés Ramírez

## Notas

- **Par front↔back**: no tiene backend propio. Consume directamente la base Supabase del producto [`sistema-ciudadano`](sistema-ciudadano.md) / [`sistema-ciudadano-bi`](sistema-ciudadano-bi.md). `baas/20260915215046_esquema.sql` es la misma migración de `sistema-ciudadano/producto/supabase/migrations/`.
- La línea gráfica "Patria" (tokens, logos de Defensores de la Patria y Banco de Talentos) está en `public/linea-grafica-patria/`.
- `baas/BASE-DE-DATOS.md` documenta las 20 tablas (18 en `participacion`, 2 en `identidad`) y un defecto conocido en el orden de creación de `esquema.sh`.
