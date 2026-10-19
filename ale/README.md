# Documentación de proyectos — carpeta `Ale`

Índice de los proyectos locales en `/Users/johnhernandez/Documents/Ale/` y de dos carpetas relacionadas en `/Users/johnhernandez/Documents/`. Cada proyecto tiene su propio archivo en esta carpeta. Fecha de relevamiento: 2026-10-19.

## Proyectos

| Nombre | Propósito | Stack | Ruta local | Doc |
| --- | --- | --- | --- | --- |
| Asistencia a talleres | Registro de asistencia a talleres por QR con calificación y panel admin (RBAC) | Next.js 16, React 19, Prisma 7, PostgreSQL 17, Tailwind 4, Docker, GitLab CI | `Ale/asistencia` | [asistencia.md](asistencia.md) |
| Mercadito (list-shopping) | App móvil offline-first de lista de mercado con historial de precios y asistente IA | Expo SDK 57, React Native 0.86, expo-sqlite, OpenAI | `Ale/projects/list-shopping` | [list-shopping.md](list-shopping.md) |
| Landing Mercadito | Landing de venta/descarga de la app | Astro 7, Tailwind 4 | `Ale/projects/landing-list-shopping` | [landing-list-shopping.md](landing-list-shopping.md) |
| Prototipo y branding Mercadito | Prototipo de diseño (Claude Design) + guía de marca; 5 copias idénticas | HTML/CSS/JS (dc-runtime) | `Ale/projects/branding/mercadito-dc-app-prototype`, `Documents/mercadito-dc-app-prototype`, `Documents/Mercadito.dc app prototype` | [mercadito-prototipo-branding.md](mercadito-prototipo-branding.md) |
| Knowledge base Mercadito | Negocio, producto, roadmap, work items, decisiones técnicas, agentes y skills | Markdown, Obsidian, Kaddo | `Ale/projects/knowledge` (+ `Ale/projects/.kaddo`) | [knowledge-mercadito.md](knowledge-mercadito.md) |
| Linux Quest (interfaz) | Juego web para aprender la terminal Linux con misiones y XP | HTML, CSS, JS vanilla | `Ale/interfaz` | [interfaz.md](interfaz.md) |
| POC prueba_velocidadquerys | Medir una consulta SQL con muchos JOIN sobre terceros en PostgreSQL | NestJS 12, pg | `Ale/pruebas_de_concepto/prueba_velocidadquerys` | [prueba-velocidadquerys.md](prueba-velocidadquerys.md) |
| POC conexion-gcp-bidquery | Conectar NestJS a Google BigQuery (ADC) y exponer un reporte | NestJS 12, @google-cloud/bigquery | `Ale/pruebas_de_concepto/conexion-gcp-bidquery` | [conexion-gcp-bidquery.md](conexion-gcp-bidquery.md) |
| Juguetería "Mundo Mágico" | Ejercicio: calcular el peso de un pedido | Python 3 | `Ale/ejercicios/jugueteria` | [jugueteria.md](jugueteria.md) |
| draw-io + diagramas BALU | Skill de Claude Code para diagramas draw.io + diagramas de arquitectura de reportería BALU (AWS) | draw.io XML, Bash, Python | `Ale/diagramas` | [draw-io.md](draw-io.md) |

## Cómo se relacionan

```
                 ┌──────────────────────────────┐
                 │ knowledge (Obsidian + Kaddo) │  negocio, producto, roadmap,
                 │  módulos: aplicacion-mobile, │  WI-001..WI-036, ADRs
                 │           landing-list-shop. │
                 └──────────────┬───────────────┘
                     documenta  │
          ┌─────────────────────┴─────────────────────┐
          ▼                                           ▼
 list-shopping (app Expo)  ◄── promociona ──  landing-list-shopping (Astro)
          ▲                                           ▲
          └──────────── branding / prototipo ─────────┘
            (paleta #00846F, tipografía, tokens, pantallas)
```

- **Producto Mercadito** (`Ale/projects/`): monorepo lógico de la app `list-shopping`, la landing `landing-list-shopping`, el diseño en `branding/` y la documentación en `knowledge/`. La app y la landing son repos git independientes sin remoto; `branding` y `knowledge` no están versionados. El prototipo de Claude Design está copiado (idéntico) dentro de la app, la landing, `branding/` y en dos carpetas sueltas de `Documents/`.
- **asistencia**: proyecto independiente (grupo GitLab astrociencia-y-tecnologia), único con remoto y CI/CD.
- **Pruebas de concepto**: ambas son APIs NestJS 12 de contexto laboral — una para rendimiento de consultas PostgreSQL de terceros y otra para lectura de BigQuery en GCP.
- **diagramas**: arquitectura de BALU (reportería sobre afiliaciones, recaudo y digiturno en AWS), no relacionada con Mercadito.
- **interfaz** y **jugueteria**: proyectos educativos/ejercicios aislados.

## Observaciones transversales

- Ninguna variable secreta se copió en esta documentación; solo se listan nombres.
- Solo `asistencia` tiene remoto git; `list-shopping` y `landing-list-shopping` tienen gran parte del trabajo sin commit; `prueba_velocidadquerys` no tiene ningún commit.
- La app Mercadito expone la API key de OpenAI en el cliente (`EXPO_PUBLIC_IA_APIKEY`): conviene un backend/proxy antes de publicar.
- Solo `conexion-gcp-bidquery` y `prueba_velocidadquerys` tienen pruebas (las generadas por NestJS con Vitest).
