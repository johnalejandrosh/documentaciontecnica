# Landing de Mercadito (landing-list-shopping)

**Ruta local:** `/Users/johnhernandez/Documents/Ale/projects/landing-list-shopping`

## Descripción / propósito

Landing page de venta/conversión de la app **Mercadito** (ver `list-shopping.md`). Su objetivo es impulsar descargas presentando la propuesta de valor, beneficios, funcionalidades, prueba social y comparativa frente a alternativas. Sitio estático, optimizado para SEO y responsive, que reutiliza el sistema de diseño del branding de la app.

## Stack y versiones

| Componente | Versión |
| --- | --- |
| Astro | ^7.3.5 |
| Tailwind CSS (`@tailwindcss/vite`) | ^4.3.3 |
| `@astrojs/sitemap` | ^3.7.4 |
| Node requerido | >= 22.12.0 |
| Gestor de paquetes | npm |

`astro.config.mjs` declara `site: https://mercadito.app` (el dominio final aún no está definido según la knowledge base). El `package.json` conserva el nombre generado `regular-remnant`.

## Arquitectura y estructura de carpetas

```
landing-list-shopping/
├── src/pages/index.astro       # Única página: compone las secciones
├── src/layouts/Layout.astro    # <head>, SEO, Open Graph, structured data
├── src/components/             # Header, Hero, Benefits, Features, SocialProof,
│                               # Comparison, CtaFinal, Footer (Welcome = plantilla)
├── src/scripts/animations.ts   # Animaciones CSS + Intersection Observer
├── src/styles/global.css       # Tokens del design system
├── src/assets/branding/        # Brief de diseño + copia del prototipo Mercadito
└── public/                     # favicon, manifest.json, robots.txt
```

Decisiones registradas (ADR en `projects/knowledge/tech/decisions/`): ADR-LP-001 Astro como framework, ADR-LP-002 animaciones CSS nativas con Intersection Observer (respeta `prefers-reduced-motion`), ADR-LP-003 SEO transversal con structured data.

## Funcionalidades / secciones

Header con navegación y CTA "Descargar" → Hero (propuesta de valor + badges Google Play / App Store) → Beneficios → Funcionalidades con mockups → Social proof → Comparativa/diferenciadores → CTA final → Footer (términos, privacidad, contacto). Los enlaces de tiendas y legales son aún placeholders (`#`).

## Datos e integraciones

Ninguna: sitio estático, sin backend, sin APIs ni base de datos. Genera sitemap.

## Cómo ejecutarlo localmente

```bash
npm install
npm run dev        # servidor de desarrollo (AGENTS.md sugiere `astro dev --background`)
npm run build      # salida estática en dist/
npm run preview
```

## Variables de entorno

Ninguna.

## Pruebas

No hay pruebas automatizadas.

## Estado git

- Repositorio propio, **sin remoto**. Rama: `main`.
- Único commit: `88c758b` "Initial commit from Astro".
- Todo el trabajo real está sin commit: modificados `astro.config.mjs`, `package.json`, `Layout.astro`, `index.astro`, `favicon.svg`; sin rastrear los componentes de secciones, `src/assets/branding/`, `manifest.json`, `robots.txt`.

## Notas

- Corresponde a la iniciativa RM-007 / work items WI-025 a WI-032 (completados) en la knowledge base.
- `README.md` es el genérico del starter de Astro.
