# requerimientostablerossoporte

> Ruta local: `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/requerimientostablerossoporte`

## Descripción / propósito

Carpeta de **requerimientos funcionales** (no es código) para los tableros de soporte del [Tablero Proyectos Externos](tablero-proyectos-externos.md). Reúne:

1. **Prompts de análisis con IA por marca** (texto para el análisis automático que alimenta el *Informe Mensual de Mesa de Ayuda*):
   - `Core Positiva promp.txt` — Positiva Compañía de Seguros, contrato CPS 0116 de 2026, Core Financiero (herramienta Aranda). ANS por severidad (Indisponibilidad 1 h, Crítico 2 h, Alto 6 h, Estándar 12 h, Bajo 24 h), umbral >= 90 %, tipologías, criterio de escalamiento N1 → N2 → N3.
   - `Core andina promp.txt` — equivalente para Core Andina.
   - `Cymetria - Pagina WEB promp.txt` — página web Positiva (Cymetria).
2. **`Matriz_ANS_por_marca (1).xlsx`** — matriz para que cada coordinador defina las reglas de ANS de su marca y configurarlas en las **políticas SLA de Zendesk**. Hojas: *Instrucciones*, *Matriz ANS*, *Condiciones y excepciones*, *Referencia métricas*, *Listas*.

## Stack y versiones

No aplica (archivos `.txt` y `.xlsx`).

## Arquitectura y estructura de carpetas

```
Core andina promp.txt
Core Positiva promp.txt
Cymetria - Pagina WEB promp.txt
Matriz_ANS_por_marca (1).xlsx
```

## Funcionalidades / pantallas

Los prompts siguen el mismo esquema que los del tablero en `lib/ai-analysis/prompts/` (hoy existen `balu`, `docum`, `fiscalia`, `ugpp` y `base`). Core Positiva, Core Andina y Cymetria **todavía no tienen prompt en el código**: estos textos son el requerimiento para agregarlos. La matriz define ANS que se reflejan en `lib/project-legends.ts` y `lib/ai-analysis/ans-contractuales.ts`.

## Datos e integraciones

Zendesk (SLA) y el análisis IA del tablero (OpenAI).

## Cómo ejecutarlo localmente

No aplica.

## Variables de entorno

Ninguna.

## Pruebas

No aplica.

## Estado git

No versionado (solo bajo el repositorio del home, sin commits).

## Notas

Fechados el 16-oct-2026. Úsalos como fuente al ajustar los prompts o los ANS en el código del tablero.
