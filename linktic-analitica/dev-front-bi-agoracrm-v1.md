# dev-front-bi-agoracrm-v1

**Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/dev-front-bi-agoracrm-v1`
**Tipo:** Frontend · **Cliente/producto:** AgoraCRM · **Área:** BI Analytics (Linktic)

## Descripción / propósito

Será el frontend de BI (dashboards) de AgoraCRM. Va a consumir la API de solo lectura `dev-back-bi-agoracrm-v1`. **Por ahora el repo solo tiene el scaffold de `create-next-app`:** la única página es la de bienvenida de Next.js y no hay pantallas, servicios ni integración con el backend.

## Stack y versiones

| Pieza | Versión |
|---|---|
| Next.js (App Router) | 16.3.8 |
| React / React DOM | 19.2.8 |
| TypeScript | ^5 |
| Tailwind CSS (vía `@tailwindcss/postcss`) | ^4 |
| ESLint + `eslint-config-next` | ^9 / 16.3.8 |
| Gestor de paquetes | pnpm 11.20.0 (`packageManager`) |

`pnpm-workspace.yaml` solo desactiva los builds nativos de `sharp` y `unrs-resolver` (`allowBuilds: false`).

## Arquitectura y estructura de carpetas

```
src/app/
  layout.tsx     # layout raíz (fuentes Geist, metadata por defecto "Create Next App")
  page.tsx       # página de bienvenida de create-next-app
  globals.css
public/          # SVG de ejemplo de Next/Vercel
AGENTS.md        # reglas de Next 16 para agentes de IA (las genera `next dev`)
CLAUDE.md        # solo importa AGENTS.md
```

## Funcionalidades / pantallas

- `/`: página por defecto de create-next-app. Todavía no hay pantallas de negocio.

## Fuentes de datos e integraciones

- Ninguna implementada.
- Backend previsto: `dev-back-bi-agoracrm-v1` (NestJS). Su `FRONT.md` explica cómo consumirlo: base `/api/v1`, login con una contraseña compartida, JWT access/refresh con reintento ante `TOKEN_EXPIRED`, y las respuestas vienen dentro de `{ data, meta }`.

## Cómo ejecutarlo localmente

```bash
pnpm install
pnpm dev      # next dev  -> http://localhost:3000
pnpm build    # next build
pnpm start    # next start
pnpm lint     # eslint
```

## Variables de entorno

No hay. El repo no tiene archivos `.env*`.

## Pruebas

No hay pruebas ni framework de testing configurado.

## Estado git

- **Remote:** `origin` → Azure DevOps `Linktic/BI Analytics/_git/dev-front-bi-agoracrm-v1`
- **Rama actual:** `feature/firts`, sincronizada con `origin/feature/firts` y 1 commit por delante de `main`
- **Ramas:** `main`, `feature/firts` (locales y remotas)
- **Últimos commits:**
  - `4909b2e` 2026-10-05 John Hernandez: firts commit (scaffold Next.js)
  - `14cb5c8` 2026-09-29 Brayan Suarez Salazar: Added README.md (README de plantilla de Azure DevOps)
- Árbol de trabajo limpio.

### Copia en el home (`/Users/johnhernandez/dev-front-bi-agoracrm-v1`)

**No es un clon de git ni un proyecto funcional.** Es una carpeta con un solo archivo, `README.md` (con fecha del 2026-10-05), y no tiene `.git` propio: git la ve como parte del repo vacío del home (`~/.git`, sin commits). Ese README es idéntico, byte a byte, al que tenía el commit inicial `14cb5c8` del repo de Analítica (la plantilla "Introduction / Getting Started" de Azure DevOps). Parece un clon o copia anterior al scaffold que quedó huérfana. **La copia válida es la de `Documents/Linktic/Analitica`.** La del home se puede borrar sin perder nada.

## Notas

- **Par front↔back:** `dev-front-bi-agoracrm-v1` (front) ↔ `dev-back-bi-agoracrm-v1` (back, NestJS + PostgreSQL de Odoo/AgoraCRM).
- Los endpoints disponibles hoy en el back para construir pantallas son `crm-leads`, `crm-leads/sucursales` y `sale-orders`.
- El nombre de la rama `feature/firts` tiene un error tipográfico ("first").
