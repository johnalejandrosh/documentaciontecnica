# Knowledge base de Mercadito (projects/knowledge)

**Ruta local:** `/Users/johnhernandez/Documents/Ale/projects/knowledge` (vault de Obsidian; configuración Kaddo en `/Users/johnhernandez/Documents/Ale/projects/.kaddo`)

## Descripción / propósito

Base de conocimiento en Markdown (vault de Obsidian, también existe `.obsidian` en `projects/`) que documenta el producto **Mercadito**: la app móvil `list-shopping` y su landing `landing-list-shopping`. Está generada y mantenida con **Kaddo**, una CLI que prepara contexto determinista (`kaddo scan`, `kaddo context`, `kaddo guard`, `kaddo owners suggest`) y paquetes de prompts de agentes para usar en un chat LLM; Kaddo no llama a LLMs ni ejecuta git. `projects/.claude/settings.json` registra el servidor MCP `kaddo-mcp`.

Proyecto Kaddo: `projects-reporting`, estado `legacy`, estructura `multirepo`, idioma español, nivel de conocimiento por defecto K2. Módulos declarados en `.kaddo/modules.yml`: `aplicacion-mobile` (→ `list-shopping`) y `landing-list-shopping` (→ Astro).

## Qué documenta (por carpeta)

| Carpeta | Contenido |
| --- | --- |
| `knowledge.md` | Estado actual del producto: propósito, arquitectura, dominios (Listas, Items, Productos, Compra, Historial, Costos) y restricciones de app y landing. |
| `business/` | `business.md`: problema (mercado sin planificación), usuarios (comprador del hogar y miembros del hogar), objetivos de negocio y restricciones (SQLite local, uso con una mano, español, sin multiusuario, precios aproximados). |
| `product/` | `product.md`: visión offline-first y *user journeys* (armar lista, comprar en el almacén, cerrar compra). `capabilities.md`: 9 capacidades en 5 dominios con evidencia, confianza y preguntas abiertas. |
| `delivery/` | `roadmap.md`: iniciativas RM-001 a RM-007 (fundación técnica, items, modo compra, cierre, historial, productos/precios, landing), supuestos resueltos. `work-items/completed/` WI-001 a WI-032; `work-items/draft/` WI-033 a WI-036 (configurar OpenAI API, secciones dentro de lista, tab Home con dashboard analítico, chat agente IA). |
| `tech/` | `current-state.md` (stack implementado, funcionalidades, esquema SQLite v2), `codebase.md`, `system/system-context.md` (componentes, sin integraciones, topología de despliegue EAS/web), `discovery/understand.md`, `modules/` (por módulo: `module-design`, `stack`, `security`, `standards`), `decisions/` (ADR-LP-001 Astro, ADR-LP-002 animaciones CSS + Intersection Observer, ADR-LP-003 SEO con structured data). |
| `agents/` | Prompts de agentes Kaddo: business, bootstrap, capability, codebase, architecture, adr, module-context, roadmap, backlog, work-item, implementation. Con reglas operativas (nunca commit/push/merge sin confirmación humana; rama `feature/<id>-<slug>` por work item). |
| `skills/` | Skills reutilizables: adr-writing, work-item-refinement, implementation-planning, learning-capture, evidence-verification, ownership-suggestion, graph-metadata-review, module-context-refinement. |
| `inventory.md` | Salida de `kaddo scan` (no detecta stack ni pruebas desde la raíz). |

Artefactos generados en `projects/.kaddo/`: `context-pack.md/json`, `graph.json`, `graph.mmd`, `graph-hints`, `scan.json`, `config.yml`, `modules.yml`. Según el context pack: fase *Active Delivery*, 36 work items (4 en draft), cobertura de ownership 12/36.

## Stack

Markdown + Obsidian + Kaddo CLI/MCP. Sin código ejecutable.

## Cómo usarlo

Abrir `Ale/projects` (o `projects/knowledge`) como vault en Obsidian. Para regenerar contexto: `kaddo scan`, `kaddo context` desde `Ale/projects`.

## Variables de entorno / pruebas

No aplica.

## Estado git

No es un repositorio propio (queda dentro del repo git del home, sin rastrear).

## Notas

- Parte de la documentación quedó desactualizada frente al código: dice "sin APIs externas / 100% offline" y "3 tabs", mientras el código ya integra OpenAI y tiene tab Home y Configuración.
- Algunas rutas citadas en `capabilities.md` (`list_repository.ts`, `list_service.ts`) son propuestas iniciales y no coinciden con los nombres reales (`list-repository.ts`, sin capa `services` para listas).
