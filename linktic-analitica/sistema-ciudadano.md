# sistema-ciudadano

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Analitica/mvp/sistema-ciudadano`
> Tipo: full-stack (Next.js + Supabase) más método de descubrimiento · Cliente/producto: Participación Ciudadana Colombia (Patria / "Defensores de la Patria")

## Descripción / propósito

El repositorio tiene dos cosas:

1. **Un método ("Fase 1, descubrir un requerimiento construyéndolo")**: una plantilla para pasar de una entrevista a una especificación, construir un MVP que obliga a decidir, registrar vacíos, validar con personas y entregar módulos formales a un equipo de desarrollo. Lo operan comandos de Claude Code (`.claude/commands/`).
2. **El producto (`producto/`)**: el sistema formal de *Participación Ciudadana Colombia, con captura, priorización y BI territorial*. Las personas conocen la agenda de encuentros, dejan aportes (por texto o voz, con apoyo de IA opcional), confirman lo entendido y consultan su tratamiento. El equipo público revisa, agrupa en expedientes, prioriza y responde.

## Stack y versiones (producto/)

| Pieza | Versión |
|---|---|
| Node | >= 24 (obligatorio: las pruebas importan `.ts` directo) |
| Next.js (App Router, Turbopack) | ^15.5.0 |
| React | ^19.1.0 |
| TypeScript | ^5.9 (estricto, `noUncheckedIndexedAccess`) |
| Tailwind CSS | ^4.1 (sin archivo de configuración) |
| Supabase | `@supabase/supabase-js` ^2.58, `@supabase/ssr` ^0.7, Supabase CLI local en Docker |
| Zod | ^4.1 |
| UI | motion, lucide-react, sonner, vaul, qrcode, @radix-ui/colors |
| Pruebas | `node --test` + Playwright ^1.62, más scripts Python (`conteo.py`, `invariantes.py`) |

## Arquitectura y estructura

Carpetas del método en la raíz:

| Carpeta | Qué es |
|---|---|
| `metodo/` | Cómo se descubre un negocio (códigos, estilo, ciclo, entrevista, frentes). Solo lectura |
| `plantillas/` | La forma en blanco de cada documento (ADR, bitácora, dominio, especificación, hoja de ruta…) |
| `negocio/` | El negocio concreto: especificación v0 (2026-09-13), fuentes, insumos, integraciones, línea gráfica, preguntas |
| `entregable/` | Módulos y mapa de módulos para el equipo de desarrollo |
| `ejemplo/` | Un negocio de ejemplo terminado (reservas), solo como referencia |
| `decisiones/` | ADRs del método (0001 "el MVP es instrumento", 0006 "stack fijo", 0007 "la arena y la línea base", etc.) |
| `construccion/` | Hoja de ruta, tareas (T0xx), progreso, revisiones y planes del producto formal |
| `harness/` | Vistas técnicas reutilizables (vista-construccion, vista-modulos) |
| `scripts/` | `esquema.sh`, `sembrar*.sh`, `divipola.sh`, `tokens.sh`, `marca.sh`, `validar.sh`, `leer-insumos.sh`, `grafo.sh`… |
| `.claude/commands/` | `/leer`, `/descubrir`, `/preguntar`, `/especificar`, `/frente`, `/vacios`, `/validar`, `/entregar`, `/donde-voy`, `/planear-construccion`, `/despachar`, `/revisar-tarea`, `/integrar`… |

Producto (`producto/src/`), organizado por dominio:

```
app/            # rutas (ver abajo)
captura/        # recibir, voz, transcribir, lectura (segmentación determinista), lectura-ia, precisar, vocero, síntesis
revision/       # bandeja, normalizar, ubicacion, expediente, recurrencia, sintesis-en-partes
territorio/     # emparejar (DIVIPOLA)
convocatoria/   # agenda, enlaces, pieza (QR/descargables)
comprobante/    # canjear
corte/          # exportar
alerta/  gestion/  priorizacion/  datos/  producto/ (marca, tema, tokens)  harness/
supabase/schemas/  # esquema declarativo (01_pertenencia … 16_enlace, 99_acceso)
supabase/migrations/20260915215046_esquema.sql
datos/divipola/    # catálogo territorial DANE versionado
```

## Funcionalidades / pantallas

| Ruta | Función |
|---|---|
| `/` | Portada con la convocatoria y los encuentros (agenda) |
| `/participar` | Captura del aporte: formulario, micrófono (voz → transcripción) y "afinado" guiado por pasos |
| `/mis-aportes` | Consulta de aportes con comprobante |
| `/e/[enlace]` | Entrada por enlace o QR de un encuentro, con confirmación |
| `/consola`, `/consola/[aporte]` | Bandeja de revisión interna; `/consola/audio/[aporte]` (route handler) sirve la grabación |
| `/administracion` | Gestión de convocatorias y encuentros; `/administracion/pieza/[enlace]/[formato]` genera piezas descargables |
| `/construccion` | Vista del harness: estado de la construcción |

La base de datos tiene 20 tablas (18 en `participacion`, 2 en `identidad`), 10 funciones (`indicadores`, `tomar_corte`, `emitir_comprobante`, `canjear_comprobante`, `estado_de_atencion`…) y seguridad de fila en `99_acceso.sql`. El contacto personal va en una partición aparte (`identidad`), separado del dato analítico.

## Fuentes de datos e integraciones

- **Supabase local (Docker)**: `project_id` `participacion`, bloque de puertos `548xx`, Storage (depósito de grabaciones).
- **OpenRouter o Mistral (opcional, INT-01)**: separa el relato libre en partes (problema, dónde, a quiénes, desde cuándo…). Modelo por defecto: `mistralai/mistral-small-3.2-24b-instruct`. Sin llave, la captura funciona con segmentación determinista, como exige `IA-01`.
- **DANE / DIVIPOLA**: catálogo territorial versionado en `producto/datos/divipola`.
- **Vercel**: los últimos commits son ajustes de despliegue.

## Cómo ejecutarlo localmente

Requisitos: Docker, Supabase CLI y Node 24 o más.

```bash
cd producto
npm ci
supabase start                 # npm run db:arrancar
cp .env.example .env.local     # pegar valores de `supabase status`
cd .. && ./scripts/sembrar.sh  # proceso de prueba, catálogo DANE, depósito de grabaciones
./scripts/sembrar-agenda.sh    # convocatoria y encuentros
./scripts/tokens.sh && ./scripts/marca.sh
cd producto && npm run dev     # http://localhost:3100
```

Otros scripts: `npm run build`, `npm start` (puerto 3100), `npm run lint`, `npm run tipos`, `npm run db:parar`, `npm run db:esquema` (genera la migración desde `supabase/schemas/`), `npm run tokens:build`. Validación completa desde la raíz: `./scripts/validar.sh`.

## Variables de entorno (solo nombres)

- `producto/.env.example`: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`, `OPENROUTER_API_KEY`, `OPENROUTER_MODELO`, `MISTRAL_API_KEY`
- Usadas además en el código: `MISTRAL_MODELO`, `IA_URL`, `PROCESO_VIGENTE`, `SIN_IA`, `DIR_BUILD`, `CI`
- Raíz `.env.example`: `MISTRAL_API_KEY` (solo para `scripts/leer-insumos.sh --motor mistral`)

## Pruebas

- Unitarias e integración: `npm test` (`node --test --env-file=.env.local pruebas/*.test.ts`), unos 22 archivos (captura, comprobante, corte, alerta, gestión, prioridad, revisión, voz…).
- E2E: `npm run test:e2e` (Playwright; `pruebas/e2e/`: participar, consola, afinar, micrófono, QR, portada, identidad, html-valido).
- Invariantes y conteos en Python: `pruebas/invariantes.py`, `pruebas/conteo.py`.
- `./scripts/validar.sh` corre tipos, build, pruebas, recorridos y una veintena de chequeos.

## Estado git

- Remoto: `https://github.com/bi-analytics-linktic/sistema-ciudadano.git`
- Rama actual: `dev` (también `main`; árbol limpio). 82 commits.
- Últimos: `11fb1d5 error deploy vercel`, `3d1aba7 error redeploy vercel`, `56e676e Cambio de Linea grafica`, `a1afdaf Puesta en marcha del producto, y el tablero al día`, `d963393 T064 · la portada toma la línea gráfica de defensoresdelapatria.com`…
- Último commit: 2026-09-16, por John Hernandez.

### Repositorio bare `mvp/sistema-ciudadano.git`

`/Users/johnhernandez/Documents/Linktic/Analitica/mvp/sistema-ciudadano.git` es un **clon espejo bare** (`core.bare=true`, remoto `old-origin` con `mirror=true`) que se usó para **migrar** el repositorio desde `bi-analytics-linktic/sistema-ciudadano` al nuevo remoto `origin` = `https://github.com/LinkTic-BI-Analytics/sistema-ciudadano-bi.git`. Tiene las ramas `main` y `dev` hasta `11fb1d5` (más un merge de PR, `2635ab8`). No es un proyecto de trabajo y se puede ignorar o borrar una vez confirmada la migración.

## Notas

- **Continuación**: el desarrollo siguió en [`sistema-ciudadano-bi`](sistema-ciudadano-bi.md) (org `LinkTic-BI-Analytics`), que parte de `11fb1d5` y añade "Te llamamos", login de la zona interna, filtros de consola y otros cambios. Este repositorio quedó congelado el 2026-09-16.
- **Par BI**: [`dev-front-patria-milagros`](dev-front-patria-milagros.md) es el tablero territorial que lee esta misma base Supabase (misma migración `20260915215046_esquema.sql`).
- Según su LEEME, el producto **no tiene autenticación** en esta versión (pregunta P4 abierta).
- El puerto 3100 se eligió porque el 3000 lo usa `linktic-crm-server` en esta máquina.
