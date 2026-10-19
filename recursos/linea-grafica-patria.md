# Línea gráfica "Patria Milagro"

Sistema de diseño unificado para el **nuevo portal web** y el **dashboard informativo** del ecosistema "Patria Milagro". Ubicación original: [`~/Documents/linea-grafica-patria/`](../../linea-grafica-patria/).

- **Modo principal:** dark. Light es un modo secundario completo (se activa con `data-theme="light"` en `<html>`).
- **Fuentes analizadas:** bancodetalentos.com.co y el especial "Primer mes de la Patria Milagro" de presidencia.gov.co (incluye sus variables CSS oficiales). defensoresdelapatria.com se descartó porque Cloudflare bloqueó el acceso.

## Contenido de la carpeta

| Ruta | Contenido |
|---|---|
| [`LINEA-GRAFICA.md`](../../linea-grafica-patria/LINEA-GRAFICA.md) | Comparativa de los sitios, decisiones de unificación, contrastes WCAG y guía de componentes |
| [`documentacion.html`](../../linea-grafica-patria/documentacion.html) | Página "Patria Milagro — Sistema de Diseño": logotipos, colores primitivos, tokens semánticos, tipografía, forma/espacio/elevación, componentes de portal y dashboard |
| [`tokens/tokens.css`](../../linea-grafica-patria/tokens/tokens.css) | Variables CSS: primitivos + semánticos dark (por defecto) y light |
| [`tokens/tokens.json`](../../linea-grafica-patria/tokens/tokens.json) | Formato W3C Design Tokens (Figma / Tokens Studio / Style Dictionary) |
| [`tokens/tailwind-theme.css`](../../linea-grafica-patria/tokens/tailwind-theme.css) | Tema Tailwind v4 (`@theme`) conectado a los semánticos |
| `assets/` | Logos (Banco de Talentos navy/blanco, Defensores bandera/blanco, Gobierno blanco), escudo de Colombia, fondo bandera dark, iconos (usuarios, podio, estrella, tigre) |
| `capturas/` | Capturas de referencia de los sitios analizados y de la documentación en dark/light (`doc-dark.png`, `doc-light.png`) |

También hay una copia comprimida en `~/Documents/linea-grafica-patria.zip`.

## Paleta de colores (primitivos)

| Familia | Tokens clave | Origen / rol |
|---|---|---|
| Azul | `--blue-50` `#F3F6FB` … `--blue-700` `#003189` (azul oficial Presidencia), `--blue-800` `#001D54`, `--blue-850` `#0A2C46` (texto navy), `--blue-900` `#0A1A3A`, `--blue-950` `#06142A` (fondo dark) | Rampa única de 12 pasos que une los azules de ambos sitios |
| Dorado | `--gold-500` `#FFC800` (CTA), `--gold-600` `#D4AA45` (dorado premium), `--gold-800` `#86691A` (texto dorado sobre claro), más 100–400 y 700 | Dos amarillos con roles distintos |
| Rojo | `--red-500` `#D80025` (oficial, light), `--red-400` `#FF4D6A` (dark), 300 y 600 | Acento puntual, nunca protagonista |
| Neutros | `--neutral-50` `#F6F5F0` (crema), `--neutral-100` `#F3F4F6`, 300/500/600/800/900 | Grafito Presidencia + crema Banco de Talentos |
| Estado | `--green-400/500`, `--orange-400/500` | Nuevos, para el dashboard |

## Tipografía

- **Display:** `Montserrat` 700–900 (uppercase solo en display y labels), tracking `-0.03em`, interlineado `1.04`.
- **Cuerpo / UI:** `Inter` 400–800 (numerales tabulares para dashboards).
- **Cifras KPI (opcional):** `JetBrains Mono`.
- **Fallback institucional:** `Nunito Sans`.
- Escala: `--text-xs` 12px (labels) · `sm` 14 · `base` 16 · `lg` 19 · `xl` 22 · `2xl` 28 · `3xl` fluido hasta 44 · `display` fluido 40–76px.
- Google Fonts: `Montserrat:wght@700;800;900` + `Inter:wght@400;600;700;800`.

## Forma, espacio, elevación y motion

- **Radios:** `--radius-xs` 2px (tags institucionales) · `sm` 6px (botones, inputs) · `md` 10px (popovers) · `lg` 16px (cards, paneles) · `full` (pills).
- **Espaciado:** base 4 (`--space-1` 4px … `--space-24` 96px).
- **Layout:** `--container-max: 1240px`, `--container-pad: clamp(1.25rem, 4vw, 3rem)`.
- **Sombras:** en dark, glow dorado en CTA (`--glow-gold`), glow azul en activos y sombras negras profundas; en light, sombras suaves azuladas. El patrón *glass* (`rgba(255,255,255,.25)` + borde translúcido) solo para overlays sobre fotografía.
- **Motion:** `cubic-bezier(0.4, 0, 0.2, 1)`, duraciones 150 / 250 / 450 ms.

## Tokens semánticos

Los componentes usan solo semánticos (los primitivos no se usan directo):

| Grupo | Tokens | Dark → Light (resumen) |
|---|---|---|
| Fondos | `--bg-base`, `--bg-surface-1/2/3`, `--bg-hero`, `--bg-inverse` | Navy `#06142A` y superficies que aclaran el navy → crema `#F6F5F0` y blanco; el hero sigue navy en ambos |
| Texto | `--text-primary/secondary/muted/inverse/accent/link/danger/success` | `#EDF1F7` con opacidades 0.72/0.55 → navy `#0A2C46`; acento gold-500 → gold-800 |
| Bordes | `--border-subtle/default/strong/accent` | Blanco 8/14/30 % → navy 8/16/32 % |
| Acción | `--action-primary-bg/text/hover`, `--action-secondary-border/text`, `--action-ghost-hover` | Primario siempre gold-500 con texto `#001D54` |
| Estado | `--status-success/warning/danger/info` + `-bg` | Variantes 400 en dark, 500/600 en light |
| Gráficas | `--chart-1…6`, `--chart-grid`, `--chart-axis-text` | Orden fijo: dorado, azul, rojo, verde, naranja, azul claro |

Con Tailwind (`tailwind-theme.css`) se exponen utilidades como `bg-base`, `bg-surface-1`, `text-primary`, `text-accent`, `border-subtle`, `bg-action-primary`; no se necesitan clases `dark:`.

## Reglas de accesibilidad (contrastes verificados)

| Combinación | Ratio | Uso |
|---|---|---|
| `#EDF1F7` sobre `#06142A` | ~15:1 AAA | Texto principal dark |
| `#FFC800` sobre `#06142A` | ~9.5:1 AAA | Acento dark |
| `#001D54` sobre `#FFC800` | ~8.9:1 AAA | Texto del botón primario |
| `#81ADEB` sobre `#0A1A3A` | ~6.3:1 AA | Links dark |
| `#FF4D6A` sobre `#0A1A3A` | ~5.4:1 AA | Peligro dark |
| `#0A2C46` sobre `#F6F5F0` | ~10.9:1 AAA | Texto principal light |
| `#86691A` sobre `#F6F5F0` | ~4.9:1 AA | Acento dorado light |

- El amarillo **nunca** lleva texto blanco: siempre navy `#001D54`.
- En dark no se usa blanco puro para texto.

## Componentes (resumen de la guía)

- **Botones:** primario dorado con texto navy, radio 6px, padding `14px 28px`, glow dorado en dark; secundario con borde 1.5px; ghost solo texto. Variante display uppercase 12px, tracking `0.08em`.
- **Navegación:** sobre `--bg-base` con `backdrop-filter: blur(12px)`; ítem activo subrayado 2px gold-500.
- **Hero:** `--bg-hero` o foto con overlay navy; kicker con tricolor de la bandera (dorado / azul / rojo) + label uppercase.
- **Cards:** `--bg-surface-1`, radio 16px, `--shadow-card`; icono dorado sobre círculo `rgba(255,200,0,.12)`.
- **Dashboard, KPI:** cifra tabular 28–44px, label 12px uppercase, delta en pill de estado, KPI destacado con borde izquierdo dorado.
- **Dashboard, gráficas:** series `--chart-1…6` (la protagonista siempre dorada), sin grid vertical, tooltip en `--bg-surface-3`, área con gradiente 12 % → 0 %.
- **Dashboard, tablas:** header sticky, cifras alineadas a la derecha con numerales tabulares.
- **Formularios:** focus con borde gold-500 + ring dorado 3px.
- **Badges / tags:** pills con pares estado + fondo; tags institucionales con radio 2px.
- **Footer:** `--blue-950` en ambos modos, enlaces con hover dorado.

## Cómo usarlo

1. Importar `tokens/tokens.css` (o `tailwind-theme.css` en proyectos con Tailwind v4) y las fuentes de Google Fonts.
2. Construir los componentes solo con tokens semánticos.
3. Cambiar el tema con `data-theme="light"` / `"dark"` en `<html>`.
4. Para Figma o Style Dictionary, importar `tokens/tokens.json`.
