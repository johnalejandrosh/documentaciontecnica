# rdt_ms_api_erp_broadcasting

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_broadcasting`

## Descripción / propósito

Servidor de **WebSockets (Laravel Reverb)** del ERP. Ofrece los canales en tiempo real que usan otros microservicios para avisar al frontend: progreso de la importación masiva de terceros, estado de los reportes del `reporter`, etc. Es uno de los servicios **obligatorios** en `services.conf` del orquestador.

El código de aplicación es casi todo la plantilla base ("Proyecto BASE"): un `ExampleController`, `SelectorsController`, `BusinessTrustModule` y `S3Service`. Su valor real está en ejecutar `php artisan reverb:start`.

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 (imagen `php:8.2-cli-alpine` con extensiones pcntl, sockets, redis) |
| Laravel | ^11.9 (lock v11.53.1) |
| Laravel Reverb | v1.10.2 |
| Sanctum | v4.2.0 |
| fiduprevisora/core | dev-develop |
| predis / redisent | ^3.3 / ^1.10 |
| Dev | PHPUnit 11, Larastan, Pint, Telescope |

## Arquitectura y estructura de carpetas

```
app/          Console, Enums, Exceptions, ExternalModules (BusinessTrustModule), Helpers, Http,
              Models, Observers, PHPStan, Providers, Rules, Services/V1 (ExampleService, S3Service)
config/reverb.php
docker-dev/php-cli/Dockerfile   imagen usada por el orquestador → CMD reverb:start --host=0.0.0.0 --port=8080
docker/                         imagen productiva php-fpm + nginx + supervisor (php-fpm, cron, laravel-worker)
infra/                          CloudFormation (ECS, ALB, SG)
```

## Módulos / endpoints principales

| Método | Ruta | Auth |
|---|---|---|
| GET | `/example` | pública |
| GET | `/select-tables` | sanctum |
| GET | `/business_trust_external_module` | sanctum (prueba) |
| GET | `/s3/buckets` | pública (lista los buckets de S3) |
| WS | `:8080` (protocolo Pusher / Reverb) | según canal |

Los canales se definen en los micros que emiten. Por ejemplo, `thirdparty.mass-import.{userId}` en thirdparties y `reporter.report-status.user-{userId}` en reporter.

## Dependencias e integraciones

- **Redis**: el contenedor depende de `redis`. Se usa para escalado de Reverb (`REVERB_SCALING_*`) y para caché.
- **PostgreSQL**: `pgsql` y `micro_base` (plantilla).
- **AWS S3**: `S3Service` (heredado).
- **Microservicios emisores**: thirdparties y reporter (usan `BROADCAST_CONNECTION=reverb` con `REVERB_*`).

## Cómo ejecutarlo localmente

Desde el orquestador. El contenedor se llama `fidu-reverb` y expone el puerto 8080:

```bash
cd ../rdt_ms_api_erp_core && ./startup.v2.sh       # 'broadcasting' es obligatorio
docker compose -f docker-compose.v2.yml --profile broadcasting up -d
```

Modo independiente heredado del `README.md`: `cp .env.example .env && docker compose up -d --build` (nginx en 84, postgres en 5436), después `php artisan migrate --database=base_pgsql --seed` y `docker network connect rdt_ms_api_erp_core_app-network project_base_php`.

## Variables de entorno (solo nombres)

`.env.example`: `APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `API_ENVIRONMENT`, `API_EXTERNAL`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `APP_MAINTENANCE_*`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_MICRO_PGSQL_*`, `LDAP_*`, `BROADCAST_CONNECTION`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `MEMCACHED_HOST`, `CACHE_STORE`, `CACHE_DRIVER`, `REDIS_CLIENT`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `REDIS_DB`, `CACHE_PREFIX`, `MAIL_*`, `AWS_*`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `AZURE_TOKEN`, `SERVICES_INTERNAL_TOKEN`, `SERVICE_TOKEN_TTL`.
En `config/reverb.php` y en el compose: `REVERB_APP_ID`, `REVERB_APP_KEY`, `REVERB_APP_SECRET`, `REVERB_HOST`, `REVERB_PORT`, `REVERB_SCHEME`, `REVERB_ALLOWED_ORIGINS`, `REVERB_SERVER`, `REVERB_SERVER_HOST`, `REVERB_SERVER_PORT`, `REVERB_SERVER_PATH`, `REVERB_MAX_REQUEST_SIZE`, `REVERB_APP_PING_INTERVAL`, `REVERB_APP_ACTIVITY_TIMEOUT`, `REVERB_APP_MAX_CONNECTIONS`, `REVERB_APP_MAX_MESSAGE_SIZE`, `REVERB_SCALING_ENABLED`, `REVERB_SCALING_CHANNEL`, `REVERB_PULSE_INGEST_INTERVAL`, `REVERB_TELESCOPE_INGEST_INTERVAL`.

## Pruebas

Solo hay unos 4 tests (plantilla). Se ejecutan con `php artisan test` (SQLite en memoria). CI: `azure-pipeline.yaml` (develop/qa, etapa `SetupEnvironment`).

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `41dab95` (2026-07-01) Merged PR 199977: fix(security): remove hardcoded azure composer token |
| Remoto | `https://dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_broadcasting` |

Es el repo con menos actividad del grupo: los últimos cambios son de seguridad de dependencias y de autenticación de servicio.

## Notas

- `README.md` y `CHANGELOG.md` son los de la plantilla y el skeleton de Laravel.
- El supervisor de la imagen productiva (`docker/php-fpm/supervisor/supervisord.conf`) levanta php-fpm, cron y `laravel-worker`, pero no Reverb. Reverb solo arranca con la imagen `docker-dev/php-cli`. Conviene revisar cómo se despliega Reverb en AWS.
- La ruta pública `/s3/buckets` expone el listado de buckets sin autenticación. Conviene revisarla.
- Hay archivos sobrantes: `docker-dev/php-fpm/Dockerfile copy`.
