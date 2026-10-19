# supabase-local

> Ruta local: `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/supabase-local`

## Descripción / propósito

Proyecto de **Supabase CLI** para levantar en local la base del [Tablero Proyectos Externos](tablero-proyectos-externos.md): crea el esquema `tablero_proyectos_externos` (tablas, vista, funciones, permisos) en un PostgreSQL local, para desarrollar y probar el tablero y la [ETL Data Core](etl-data-core.md) sin tocar la base de Supabase en la nube. Es el destino de la cadena local que aparece en el `.env.example` del tablero (`127.0.0.1:54322`).

## Stack y versiones

- Supabase CLI (última versión detectada en `.temp/cli-latest`: v2.120.0), Docker
- PostgreSQL 17 (`[db] major_version = 17`)
- `project_id = "tablero_externos"`

## Arquitectura y estructura de carpetas

```
supabase/
  config.toml                                   configuración del stack local
  migrations/20261014000000_esquema_tablero.sql esquema completo (~1.100 líneas, 12 CREATE TABLE)
  snippets/  .branches/  .temp/                 artefactos de la CLI (no versionar .temp)
```

## Funcionalidades / servicios

| Servicio | Puerto local |
|---|---|
| API (REST, expone `public`, `graphql_public`, `tablero_proyectos_externos`) | 54321 |
| Base de datos | 54322 (shadow 54320) |
| Studio | 54323 |
| SMTP local (Inbucket/Mailpit) | 54324 |
| Pooler (desactivado) | 54329 |

Auth, Storage, Realtime habilitados; seed habilitado en `db reset`.

## Datos e integraciones

Tablas del esquema: `modules`, `profiles`, `profile_modules`, `data`, `data_core_cases`, `modulo_fuente_api`, rollups, entre otras. Los datos se cargan aparte con los archivos de `01_tablero/datos/` en el orden de `base-de-datos.md` sección 7.

## Cómo ejecutarlo localmente

```bash
cd /Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/supabase-local
supabase start          # levanta los contenedores y aplica migrations/
supabase db reset       # recrea la base desde las migraciones (borra datos locales)
supabase stop
```

## Variables de entorno

No usa `.env` propio. La CLI genera secretos de runtime en `supabase/.temp/start-secrets/` (no compartir ni versionar). Los consumidores (tablero y ETL) apuntan a la base con `DATABASE_URL`.

## Pruebas

No aplica.

## Estado git

No es un repositorio propio (queda bajo el repositorio del home, sin commits). Tiene `supabase/.gitignore` de la CLI.

## Notas

El tablero ya no depende de la API ni del Auth de Supabase (solo de PostgreSQL), así que basta con el contenedor de la base; el resto de servicios es opcional.
