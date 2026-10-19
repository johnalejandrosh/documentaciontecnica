# POC: conexion-gcp-bidquery (NestJS + BigQuery)

**Ruta local:** `/Users/johnhernandez/Documents/Ale/pruebas_de_concepto/conexion-gcp-bidquery` (existe además un `conexion-gcp-bidquery.zip` en la carpeta padre)

## Descripción / propósito

Prueba de concepto de conexión de una API NestJS a **Google BigQuery** usando credenciales ADC (gcloud). Expone un endpoint que consulta una tabla de reporte de un dataset de desarrollo (`ambiente_dev`) del proyecto GCP de Positiva (`positiva-sgda-prd-416417`), tabla `tb_reporte_Medicina_Entrada_28052026`, con `LIMIT 100`.

## Stack y versiones

NestJS ^12.0.1, `@google-cloud/bigquery` ^9.1.0, `@nestjs/observe` ^0.1.8 (observabilidad, con claves placeholder), TypeScript ^6.0.2, ESM, Vitest ^4.1.2, oxlint, pnpm.

## Estructura

- `src/bigquery/bigquery.service.ts`: cliente `BigQuery` con `projectId` desde `GCP_PROJECT_ID` (con valor por defecto en código) y método `runQuery`.
- `src/bigquery/bigquery.controller.ts`: endpoint de consulta.
- `.claude/skills/nestjs-bigquery/SKILL.md`: skill de Claude Code (`/nestjs-bigquery`) que automatiza esta misma configuración (instalar dependencia, generar service/controller/tests, verificar ADC y projectId, resolver errores comunes).

## Endpoints

- `GET /bigquery/reporte` — devuelve hasta 100 filas de la tabla de reporte.
- `GET /` — Hello World del starter.

## Cómo ejecutarlo

```bash
gcloud auth application-default login   # credenciales ADC
pnpm install
pnpm run start:dev                       # puerto PORT o 3000
```

## Variables de entorno (solo nombres)

`GCP_PROJECT_ID` (opcional), `PORT` (opcional). Credenciales vía ADC (sin archivo `.env`).

## Pruebas

`bigquery.service.spec.ts`, `bigquery.controller.spec.ts`, `app.controller.spec.ts` y e2e con Vitest (`pnpm test`, `pnpm test:e2e`).

## Estado git

Repositorio propio, rama `main`, sin remoto. Único commit: `7f5f645` "conexion a GCP". Árbol limpio.

## Notas

- La carpeta se llama "bidquery" (errata de "bigquery").
- `ObserveModule` tiene `appKey`/`appSecret` como placeholders (`YOUR_APP_KEY`); no hay credenciales reales en el código.
- README genérico de NestJS.
