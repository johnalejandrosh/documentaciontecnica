# POC: prueba_velocidadquerys

**Ruta local:** `/Users/johnhernandez/Documents/Ale/pruebas_de_concepto/prueba_velocidadquerys`

## Descripción / propósito

Prueba de concepto en NestJS para medir la velocidad de una consulta SQL directa (con `pg`, sin ORM) sobre un modelo de **terceros** (`third_parties`) en PostgreSQL. La consulta reemplaza un `include` anidado típico de ORM por un único `SELECT` con ~20 `LEFT JOIN` (tipo de documento, estado, persona natural/jurídica, fideicomiso, representantes legales, direcciones, contactos, cuentas bancarias, actividades económicas/CIIU, info laboral, tributaria, financiera, PEP, documentos, tipos de tercero e identidad), filtrando terceros que no son clientes, ni fideicomisos, ni auxiliares contables.

## Stack y versiones

NestJS ^12.0.1, `@nestjs/config` ^4.0.4, `pg` ^8.23.0, TypeScript ^6.0.2, ESM (`"type": "module"`), Vitest ^4.1.2, oxlint, Prettier, pnpm.

## Estructura

- `src/database/`: provider `DATABASE_POOL` (`pg.Pool`, máx. 20 conexiones, SSL según `DB_SSL` o `NODE_ENV`) y `DatabaseService` (`query`, `getClient`, `transaction`).
- `src/terceros/`: módulo con controller, service, DTOs y entidad vacía.

## Endpoints

Controller montado en `/terceros` y `/third-parties`:
- `GET /terceros` — ejecuta la consulta y devuelve `{ rowCount, data }`. **Único endpoint implementado.**
- `POST`, `GET /:id`, `PATCH /:id`, `DELETE /:id` — stubs del generador (devuelven texto).

## Cómo ejecutarlo

```bash
pnpm install
pnpm run start:dev     # puerto PORT o 3000
```

## Variables de entorno (solo nombres)

`NODE_ENV`, `PORT`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSL`, `SHEMA` (definida en `.env` pero no usada en el código).

## Pruebas

Specs generados (`*.spec.ts`) y `test/app.e2e-spec.ts` con Vitest (`pnpm test`, `pnpm test:e2e`); no prueban la consulta real. No hay medición de tiempos en código: la velocidad se mide externamente (cliente HTTP).

## Estado git

Repositorio inicializado en rama `main` **sin ningún commit** y sin remoto; todos los archivos sin rastrear (`.env` ignorado).

## Notas

README genérico de NestJS.
