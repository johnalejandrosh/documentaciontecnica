# sistema-ciudadano-bi

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/mvp/sistema-ciudadano-bi`
> Tipo: full-stack (Next.js + Supabase) · Cliente/producto: Participación Ciudadana Colombia (Patria). Nombre del sistema desde 2026-09-21: **"Sistema de Escucha y Planeación Nacional"**

## Descripción / propósito

Es la **continuación** de [`sistema-ciudadano`](sistema-ciudadano.md), migrada a la organización `LinkTic-BI-Analytics`. Comparte el mismo método (metodo/, plantillas/, negocio/, decisiones/, construccion/, comandos de Claude) y el mismo producto en `producto/`: captura de aportes ciudadanos (texto, voz, IA opcional), revisión en consola, expedientes, priorización, alertas y agenda de encuentros. Sobre la base `11fb1d5` agrega:

- **"Te llamamos" (T066)**: la persona deja nombre y teléfono para que la llamen (`src/llamada/`, esquema `17_llamada.sql` en la partición `identidad`) y se avisa por un webhook.
- **Login de la zona interna**: `/ingresar` con token de acceso y JWT, y un `middleware.ts` que protege `/consola` y `/administracion` con cookie de acceso y de renovación.
- **Consola mejorada**: filtros por departamento y municipio, paginación, resumen, filtros activos, estados de carga y error.
- **"¿Desde dónde nos contactas?"** al final de la captura, con soporte de países (`territorio/paises.ts`).
- Siembra de agenda con el cronograma oficial (`convocatoria/cronograma.ts`, `sembrar-agenda.sh` con soporte para base externa), nuevo armazón con barra lateral y tokens propios (ciudadano e interno).
- `plan_trabajo.md`: plan de refactor (p. ej. `afinado.tsx` con 1.033 líneas).

## Stack y versiones (producto/)

Igual que `sistema-ciudadano`: Node >= 24, Next.js ^15.5 (Turbopack), React ^19.1, TypeScript ^5.9, Tailwind ^4.1, Supabase (`supabase-js` ^2.58, `ssr` ^0.7, CLI local en Docker), Zod ^4.1, Playwright ^1.62, motion, lucide-react, sonner, vaul, qrcode. Hay `package-lock.json` y también `pnpm-lock.yaml`/`pnpm-workspace.yaml`.

## Arquitectura y estructura

La misma partición del método (ver [sistema-ciudadano.md](sistema-ciudadano.md)), más `graphify-out/` (grafo de conocimiento), `.obsidian/` y `plan_trabajo.md`. En `producto/src/` se suman `acceso/` (sesión y JWT), `llamada/` (aviso, registrar), `middleware.ts`, `datos/leer.ts` y `producto/` (armazón, barra lateral, nombre, sectores, iconos, esqueleto). El esquema queda en la migración `20260921162435_esquema.sql`.

## Funcionalidades / pantallas

| Ruta | Función |
|---|---|
| `/` | Portada con agenda y calendario de encuentros |
| `/participar` | Captura: formulario, micrófono, afinado por pasos, progreso, comprobante, "Te llamamos" |
| `/mis-aportes` | Consulta con comprobante |
| `/e/[enlace]` | Entrada por enlace o QR de encuentro |
| `/ingresar` | Login de la zona interna (token de acceso) |
| `/consola`, `/consola/[aporte]` | Bandeja de revisión con filtros territoriales, paginación y resumen (protegida) |
| `/consola/audio/[aporte]` | Audio de la grabación (route handler) |
| `/administracion`, `/administracion/pieza/[enlace]/[formato]` | Gestión de convocatorias y piezas descargables (protegida) |
| `/construccion` | Vista de estado de construcción (harness) |

## Fuentes de datos e integraciones

- **Supabase/Postgres**: esquemas `participacion` e `identidad` (más la tabla de llamadas), Storage para grabaciones. Local en Docker (`project_id` `participacion`, puertos `548xx`) o base externa.
- **OpenRouter o Mistral (opcional)**: corte y estructuración del relato capturado.
- **Webhook de llamadas** (`LLAMADA_WEBHOOK_URL`): avisa de las solicitudes "Te llamamos"; se desactiva con `SIN_WEBHOOK`.
- **DANE / DIVIPOLA** y catálogo de países.
- **Vercel** como destino de despliegue.

## Cómo ejecutarlo localmente

Igual que `sistema-ciudadano`:

```bash
cd producto && npm ci && supabase start
cp .env.example .env.local      # valores de `supabase status` + tokens
cd .. && ./scripts/sembrar.sh && ./scripts/sembrar-agenda.sh && ./scripts/tokens.sh && ./scripts/marca.sh
cd producto && npm run dev      # http://localhost:3100
```

Scripts: `dev`, `build`, `start`, `lint`, `tipos`, `test`, `test:e2e`, `db:arrancar`, `db:parar`, `db:esquema`, `tokens:build`. Desde la raíz también: `scripts/validar.sh`, `scripts/paises.sh`, `scripts/migrar-contacto.sh`, `scripts/sembrar-bandeja.sh`.

## Variables de entorno (solo nombres)

- `producto/.env.example`: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`, `OPENROUTER_API_KEY`, `OPENROUTER_MODELO`, `MISTRAL_API_KEY`, `LLAMADA_WEBHOOK_URL`, `SIN_WEBHOOK`, `ACCESS_TOKEN`, `JWT_SECRET`
- Usadas además en el código: `MISTRAL_MODELO`, `IA_URL`, `PROCESO_VIGENTE`, `SIN_IA`, `DIR_BUILD`, `CI`, `NODE_ENV`
- Sin `ACCESS_TOKEN` o sin `JWT_SECRET`, la zona interna queda cerrada para todos (diseño intencional).

## Pruebas

`npm test` (node --test, unos 30 archivos en `producto/pruebas/`, incluida `llamada.test.ts`), `npm run test:e2e` (Playwright: participar, consola, afinar, identidad, QR…), invariantes en Python y `scripts/validar.sh` (19 chequeos).

## Estado git

- Remoto: `https://github.com/LinkTic-BI-Analytics/sistema-ciudadano-bi.git`
- Rama actual: `feature/bi` (árbol limpio). Locales: `main`, `feature/form`, `feature/tellamamos`. Remotas también: `dev`, `feature/paises`. 96 commits.
- Últimos: `5aaf74d implementacion de token de acceso al panel de administracion, cambio de nombre`, `74bf21e feat: «¿desde dónde nos contactas?» al final de la captura`, `3298639 feat: enhance sembrar-agenda.sh to support external database connections`, `5b97e8d siembra de encuentros en la agenda…`, `8d18669 Merge PR #4 feature/tellamamos`…
- Último commit: 2026-09-21, por John Hernandez.

## Notas

- **Origen**: es un fork o migración de `bi-analytics-linktic/sistema-ciudadano`. El bare `mvp/sistema-ciudadano.git` se usó como espejo para esa migración (ver [sistema-ciudadano.md](sistema-ciudadano.md)).
- **Par BI**: [`dev-front-patria-milagros`](dev-front-patria-milagros.md) lee la misma base Supabase para el tablero territorial. Ojo: patria-milagros trae la migración vieja (`20260915215046`) y este repositorio ya está en `20260921162435`.
- Hay un `.pyc` versionado por error (`scripts/lib/__pycache__/`), y `plan_trabajo.md` deja abiertos Q35 ("Te llamamos" con gente real) y P4 (identidad).
