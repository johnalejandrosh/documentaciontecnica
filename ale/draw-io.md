# Diagramas y skill draw-io

**Rutas locales:**
- Skill: `/Users/johnhernandez/Documents/Ale/diagramas/draw-io`
- Diagramas: `/Users/johnhernandez/Documents/Ale/diagramas/*.drawio`

## Descripción / propósito

1. **`draw-io/`** es una *skill* de Claude Code (de terceros; referencias en japonés) para crear, editar y revisar diagramas `.drawio` como XML, con reglas de fuentes, layout, iconos oficiales de AWS y conversión automática a PNG.
2. Los archivos `.drawio` de la carpeta padre son diagramas de arquitectura del sistema de reportería **BALU** creados con esa skill.

## Contenido de la skill

| Archivo | Función |
| --- | --- |
| `SKILL.md` | Instrucciones: editar solo `.drawio` (nunca `.drawio.png`), `defaultFontFamily`/`fontFamily` (p. ej. Noto Sans JP para slides Quarto), comandos de conversión, reglas de layout. |
| `README.md` | Descripción en inglés: propósito, cuándo usarla, características (fuentes, conversión PNG, cálculo de layout, iconos AWS, QA). |
| `references/aws-icons.md` | Nombres oficiales de servicios AWS y estilos de iconos (japonés). |
| `references/layout-guidelines.md` | Guía de agrupación y jerarquía (AWS Cloud → VPC → subredes), espaciado (japonés). |
| `scripts/convert-drawio-to-png.sh` | Exporta cada `.drawio` a `.drawio.png` (escala 2x, fondo transparente) con la CLI `drawio` y hace `git add` de los PNG (pensado como hook de pre-commit). |
| `scripts/find_aws_icon.py` | Busca un servicio AWS en `aws-icons.md` y devuelve su estilo de icono: `python find_aws_icon.py lambda`. |

## Diagramas BALU

| Archivo | Contenido |
| --- | --- |
| `balu-aws.drawio` | Infraestructura AWS: CloudFront + S3 (SPA Reporteador y SPA Dashboards), API Gateway + VPC Link, ALB interno, ECS Fargate (back-ends Afiliaciones, Recaudo, Digiturno), RDS PostgreSQL (Afiliaciones, Recaudo), Lambda constructora de archivos, S3 de archivos generados, NAT/Internet Gateway, proveedor externo Supabase (Digiturno). |
| `balu-componentes.drawio` | Componentes: rutas `/afiliaciones/*`, `/recaudo/*`, `/digiturno/*`, `POST /reportes`; capas controller → servicios → repositorio; generador CSV/XLSX; cliente S3 (PutObject + URL prefirmada); Secrets Manager, IAM, CloudWatch. |
| `balu-flujo-reportes.drawio` | Flujo asíncrono: solicitud → SQS → worker → lectura de Afiliaciones/Recaudo → CSV a S3 → notificación por API Gateway WebSocket → descarga con URL prefirmada; Digiturno síncrono. |
| `producto.drawio` | Vista de producto "Producto Reportería": UI de reportes, API REST y bases PostgreSQL afiliaciones, digiturno y recaudo. |

## Stack / requisitos

draw.io (app de escritorio o CLI `drawio`), Bash, Python 3; opcionalmente `mise` + `pre-commit`.

## Cómo usarlo

Abrir los `.drawio` con draw.io / diagrams.net. Convertir a PNG: `bash draw-io/scripts/convert-drawio-to-png.sh balu-aws.drawio` (requiere la CLI `drawio`; ojo, el script hace `git add`).

## Variables de entorno / pruebas

No aplica.

## Estado git

No es un repositorio propio (queda dentro del repo git del home, sin rastrear).
