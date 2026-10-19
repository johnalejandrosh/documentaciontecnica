# Mercadito — prototipo de diseño y branding

Documento combinado para todas las copias del prototipo de diseño de Mercadito exportado desde Claude Design.

## Rutas locales

| Ruta | Contenido |
| --- | --- |
| `/Users/johnhernandez/Documents/Ale/projects/branding/mercadito-dc-app-prototype` | `README.md` + `project/` (copia de referencia del monorepo Mercadito) |
| `/Users/johnhernandez/Documents/mercadito-dc-app-prototype` | `README.md` + `project/` (bundle de handoff tal como se exportó) |
| `/Users/johnhernandez/Documents/Mercadito.dc app prototype` | Solo los archivos del proyecto (`Mercadito.dc.html`, `support.js`, `.thumbnail`) sin README |
| `/Users/johnhernandez/Documents/Ale/projects/list-shopping/assets/branding/` | Otra copia del bundle + `context_diseno.md` |
| `/Users/johnhernandez/Documents/Ale/projects/landing-list-shopping/src/assets/branding/` | Otra copia del bundle + `context_diseno.md` |

**Comparación:** se verificó por checksum MD5 que `Mercadito.dc.html`, `support.js`, `.thumbnail` y `README.md` son **idénticos byte a byte** en las cinco ubicaciones. Es el mismo prototipo duplicado; `Ale/projects/branding/` es la copia que la knowledge base toma como fuente del sistema de diseño. `context_diseno.md` (idéntico en app y landing) solo existe junto a esos dos proyectos.

## Descripción / propósito

*Handoff bundle* de Claude Design (claude.ai/design): un prototipo HTML/CSS/JS de la app Mercadito pensado para que un agente de código lo recree "pixel-perfect" en la tecnología destino (Expo/React Native en `list-shopping`, Astro en la landing). No es código de producción.

`context_diseno.md` es el brief de diseño que originó el prototipo: qué es la app, problema, usuario objetivo (compradores del hogar en Colombia, 25-55 años, uso con una mano), personalidad de marca (práctica, cálida, confiable, ágil), tono (español colombiano, tuteo) y requisitos de nombre/logo.

## Stack

- HTML estático + `support.js` (runtime "dc-runtime" generado, ~69 KB, que usa React global y una sintaxis de plantillas `<x-dc>`, `<sc-if>`, `{{ }}`).
- `.thumbnail`: imagen WebP de vista previa.
- `Mercadito.dc.html`: ~59 KB.

## Contenido del prototipo

Dos pantallas/lienzos (`data-screen-label`):

1. **App** — maqueta de iPhone (390×844) con estados: Lista ("Mi Lista", total estimado, "Toca para marcar", comprados / no comprados), edición/nuevo item (nombre, cantidad, unidad, imprevisto "se suma directo al carrito"), cerrar compra ("¿Cuánto pagaste en total?", estimado vs. real, "¡Listo! Tu compra quedó guardada 🎉") e Historial ("Tus compras pasadas", estados vacíos).
2. **Guía de marca v1** — paleta clara y oscura, tipografía system-ui (SF Pro / Roboto) con escala (Display 34-36, Title 32, Subtitle 22, Body 17, Body small 14, Caption 12), espaciado, radios, movimiento, toque, ícono (incluye Android adaptativo), tokens y guía de tono ("Así hablamos / Así no").

Color principal: verde `#00846F` (hover `#00574A`), fondo `#E7E3DA`, texto `#1C1B18`; mismo verde usado en el splash y el ícono adaptativo de la app.

## Cómo verlo localmente

Abrir `project/Mercadito.dc.html` en un navegador (carga `./support.js` relativo). El README del bundle recomienda a los agentes leer el HTML en lugar de renderizarlo.

## Variables de entorno / pruebas

No aplica.

## Estado git

Ninguna de las ubicaciones es un repositorio propio (quedan dentro del repositorio git del home de usuario, sin rastrear). Las copias dentro de `list-shopping` y `landing-list-shopping` están sin commit en sus respectivos repos.

## Notas

- Recomendación: conservar una sola copia canónica (p. ej. `Ale/projects/branding/`) y eliminar o enlazar las demás para evitar divergencias.
