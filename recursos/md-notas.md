# Notas Markdown: Balú y Fidu

Notas de trabajo en Markdown guardadas fuera de los repositorios, en [`~/Documents/md/`](../../md/), más [`~/Documents/README_BALU.md`](../../README_BALU.md) en la raíz de Documents. Son copias de los archivos `CLAUDE.md`, `FRONT.md` y `README.md` de dos backends BI de LinkTic: el tablero **Balú** (ARL Positiva) y los servicios BI de **Fiduprevisora**. Hay una copia comprimida en `~/Documents/md_projects.zip`. Esa copia incluye además `md/Fidu/README_FIDU.md`, que ya no está en la carpeta.

| Proyecto | Archivo | Líneas | Tipo |
|---|---|---|---|
| Balú | [`md/Balu/CLAUDE_BALU_AFILIACIONES.md`](../../md/Balu/CLAUDE_BALU_AFILIACIONES.md) | ~350 | Guía para Claude Code (CLAUDE.md) |
| Balú | [`md/Balu/FRONT_BALU_AFILIACIONES.md`](../../md/Balu/FRONT_BALU_AFILIACIONES.md) | ~360 | Contrato Front ↔ Back (FRONT.md) |
| Balú | [`README_BALU.md`](../../README_BALU.md) | ~115 | README del repo `bi-back-balu-board` |
| Fidu | [`md/Fidu/CLAUDE_FIDU.md`](../../md/Fidu/CLAUDE_FIDU.md) | ~450 | Guía para Claude Code (CLAUDE.md) |
| Fidu | [`md/Fidu/FRONT_FIDU.md`](../../md/Fidu/FRONT_FIDU.md) | ~6.800 | Contrato de API para el Front (`rdt-ms-reporter-bi`) |

---

## Balú: tablero de Afiliaciones (ARL Positiva)

### `README_BALU.md`: BI Back Balu Board
README del backend **de solo lectura** (NestJS + `pg`, sin ORM) que lee de una **réplica de lectura de PostgreSQL**. Incluye:
- Índice de la documentación del repo (`docs/es/01…13`): configuración, arquitectura, seguridad, JWT, base de datos, endpoints, caché Redis (*stale-while-revalidate*, encabezado `X-Cache`), logs, rendimiento (diagnóstico de los 504, `GROUPING SETS`), comandos de consola, los 28 constructores de SQL e índices de la réplica.
- Puesta en marcha (Node ≥ 22, pnpm, puerto 3001, prefijo `/api/v1`, Swagger en `/docs` solo con `NODE_ENV=local`).
- Autenticación por **rol** (Digiturno / Afiliaciones / Gerencia) con un token por rol definido en el entorno; emite JWT de acceso (~15 min) y refresh (~12 h) con rotación y revocación.
- Tabla de endpoints (`auth/*`, `afiliaciones/resumen`, `afiliaciones/empresas`, `afiliaciones/afiliados`, `cache/clear-cache`, `/health`), los scripts pnpm y Docker.
- Advierte que `POST /api/v1/cache/clear-cache` es público y puede degradar el servicio.

### `CLAUDE_BALU_AFILIACIONES.md`
Guía para que Claude Code trabaje en el repo del back de Afiliaciones:
- **Qué es el proyecto**: API de solo lectura que sirve el tablero de Afiliaciones del aplicativo Balú. `FRONT.md` manda sobre la forma de las respuestas. Incluye el estado de la base medido en la réplica real.
- **Reglas críticas**: solo `SELECT`/`WITH` (validado por `assertReadOnlySql` y sesiones `read_only`); solo 5 fuentes autorizadas (`rep_affiliate`, `rep_affiliate_people`, `rep_employer`, `rep_sede`, `rep_divipola`); uniones por `filed_number` (excepto la resolución geográfica DANE por `divipola`); esquema configurable (default `bi`); SQL siempre parametrizado; `process.env` solo vía `EnvConfigService`; rutas protegidas por defecto + `RolesGuard`; contrato del Front intocable.
- **Arquitectura**: módulos, capas, caché y consultas.
- **Tests**: 934 unitarios + 28 e2e, ninguno necesita BD.
- Comandos, documentación y entorno de desarrollo.

### `FRONT_BALU_AFILIACIONES.md`
Contrato de endpoints que el **Front** (`front-bi-balu.linktic.com`, equipo BI Analytics) le pide al **Back**, sin SQL:
- Balú tiene dos tableros: **Digiturno** (lee Supabase directamente, sin back) y **Afiliaciones** (todo viene de este back).
- §0 Convenciones · §1 Autenticación JWT (`login`, `refresh`, `logout`, `me`, notas de seguridad) · §2 Tablero de Afiliaciones: filtros comunes (`desde`, `hasta`, `estado`, `categoria`, `departamento`, `municipio`, `riesgo`, `sector`), `GET /afiliaciones/resumen` (payload agregado), `/empresas` y `/afiliados` (paginados) · §3 Contrato de datos geográficos para el mapa (crítico) · §4 Resumen de endpoints · §5 Qué hace el Front mientras tanto.

---

## Fidu: servicios BI de Fiduprevisora

### `CLAUDE_FIDU.md`
Guía para Claude Code en el repo base de servicios BI de Fiduprevisora (NestJS 11 + `pg`, solo lectura):
- Nació como infraestructura común (configuración, datos, caché, logs, salud). El módulo de Afiliaciones y la autenticación por roles se **eliminaron**.
- Lista todos los endpoints de reportes bajo `/api/v1` (terceros, transacciones fiduciarias, embargos, conciliaciones, portal de pagos, comisiones, FIC, tributario, presupuesto, cuentas por pagar).
- **Reglas críticas**: solo lectura, sin inyección SQL (allowlists para `sort`), esquema configurable (default `bi`) con excepciones por conexión para `tax-related` (`tax_service_v2`), `budget` y `accounts-payable` (`public`), y `process.env` solo vía `EnvConfigService`.
- Arquitectura (la sección más extensa), documentación y entorno de desarrollo.

### `FRONT_FIDU.md`: contrato de API `rdt-ms-reporter-bi`
La fuente de verdad del contrato para el cliente web (~6.800 líneas):
- **Premisas**: solo lectura; **sin autenticación** (el control de acceso está en la red, el proxy o el gateway); devuelve lo que produce PostgreSQL sin transformar (claves `snake_case`).
- **Índice de 35 endpoints `GET` de datos + 2 de operación** con su `query_id`, su granularidad y la forma de la respuesta `{ count, module_name, report_name, query_id, data }`.
- Secciones por módulo: §2 Terceros · §3 Embargos · §4 Conciliaciones bancarias · §5 Solicitudes de pago del portal · §6 Transacciones fiduciarias (listado maestro, garantías) · §7 Liquidación de comisiones · §8 FIC (planes, operaciones, movimientos, cierres) · §9 Tributario (impuestos, certificados, cierres, formularios DIAN, historial) · §10 Presupuesto (7 vistas materializadas) · §11 Cuentas por pagar (8 reportes).
- Transversales: §1 Convenciones · §12 Tipos en JSON (las dos trampas de tipos que causan la mayoría de los bugs de integración) · §13 Errores · §14 Operación (`/health`, `clear-cache`) · §15 Referencia rápida de params por endpoint · "Estado de este documento" (bitácora de cambios, p. ej. el envelope nuevo de `/thirdparties` del 2026-09-03 y Presupuesto del 2026-09-08).
- Notas clave: casi ningún endpoint acepta filtros (cualquier param da `400`); solo Terceros y Transacciones fiduciarias paginan (`page`/`pageSize`).
