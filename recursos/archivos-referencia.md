# Archivos de referencia sueltos en `~/Documents`

Inventario de los archivos que están sueltos en la raíz de `~/Documents`. Para cada uno se dice de qué trata y a qué proyecto pertenece. El contenido se dedujo del nombre y, en los `.zip`, del listado de su contenido (`unzip -l`, sin extraer). Las rutas son relativas a `~/Documents/`.

| Archivo | Tipo | De qué trata | Proyecto relacionado |
|---|---|---|---|
| `ANEXO 06. ANEXO TÉCNICO (2) (1).pdf` | PDF (37 págs., ~640 KB) | Anexo técnico n.º 6 de un proceso o contrato: requisitos y especificaciones técnicas | Por confirmar (probablemente una licitación o propuesta de LinkTic) |
| `My Estimate - Calculadora de precios de AWS.pdf` | PDF (2 págs.) | Estimación de costos exportada desde AWS Pricing Calculator | Infraestructura AWS de un proyecto BI / LinkTic |
| `Catálogo de integración para el generador dinámico de reportes.xlsx` | Excel (~310 KB) | Catálogo de fuentes, vistas y consultas para integrar con el generador dinámico de reportes | Fiduprevisora BI (`rdt-ms-reporter-bi`) / `backend-reporting-template` |
| `balu-afiliaciones_dependientes_2024-01-01_2026-10-07.xlsx` | Excel (~1,2 MB) | Exportación de afiliaciones de **trabajadores dependientes** del 2024-01-01 al 2026-10-07 | Balú: tablero de Afiliaciones (ARL Positiva) |
| `balu-afiliaciones_servicio-domestico_2024-01-01_2026-10-07.xlsx` | Excel (~180 KB) | Exportación de afiliaciones de **servicio doméstico** en el mismo rango | Balú: Afiliaciones |
| `balu-digiturno_afiliacion_2026-10-01_2026-10-07.xlsx` | Excel (~60 KB) | Exportación de turnos de Digiturno del módulo de afiliación (1 al 7 de octubre de 2026) | Balú: tablero Digiturno |
| `md_projects.zip` | ZIP (~32 KB) | Respaldo de la carpeta `md/`: `Balu/CLAUDE_BALU_AFILIACIONES.md`, `Balu/FRONT_BALU_AFILIACIONES.md`, `Fidu/CLAUDE_FIDU.md` y `Fidu/README_FIDU.md` (este último ya no está en la carpeta) | Balú / Fidu (ver [md-notas.md](md-notas.md)) |
| `backend-reporting-template.zip` | ZIP (~11 KB) | Plantilla NestJS de backend de reportes: `ARCHITECTURE.md`, `FRONT.md`, `README.md`, configuración (eslint, tsconfig, nest-cli), módulo `infrastructure/database`, módulo `example-report` (controller/service/repository/sql), `health` y tests e2e. Existe también la carpeta `backend-reporting-template/` | Plantilla base para backends BI de LinkTic |
| `Mobile app design brief.zip` | ZIP (~44 KB) | Prototipo HTML de diseño: `Health Apps.dc.html` + `support.js` + miniatura | Brief de diseño de app móvil (apps de salud) |
| `Mercadito.dc app prototype.zip` | ZIP (~54 KB) | Prototipo HTML `Mercadito.dc.html` + `support.js` + miniatura | Mercadito (prototipo de app) |
| `Mercadito.dc app prototype-handoff.zip` | ZIP (~56 KB) | El mismo prototipo empaquetado para handoff (`mercadito-dc-app-prototype/README.md` + `project/`). Coincide con la carpeta `mercadito-dc-app-prototype/` | Mercadito |
| `linea-grafica-patria.zip` | ZIP (~8,6 MB) | Respaldo de la carpeta `linea-grafica-patria/` (MD, HTML, tokens, assets y capturas) | Patria Milagro (ver [linea-grafica-patria.md](linea-grafica-patria.md)) |
| `README_BALU.md` | Markdown | README del backend `bi-back-balu-board` | Balú (ver [md-notas.md](md-notas.md)) |
| `scriptstui` | Binario Mach-O arm64 (~5 MB) | Ejecutable de una herramienta TUI de scripts compilada localmente | Herramientas personales |

## Archivos excluidos a propósito

- **Bases de contraseñas KeePass (`.kdbx`)**: `AccesosAnalitica.kdbx` y `AccesosAnaliticaobsoleto.kdbx`. Guardan los accesos del área de analítica y una versión obsoleta. **No se abrieron ni se documenta su contenido.**
- **Archivos de entorno** (`.env`, `env`): pueden contener secretos. No se leyeron.
- **Instaladores `.dmg`** (Ghostty, LibreOffice): solo son software de instalación.
- Archivos del sistema (`.DS_Store`, `.localized`).
