# rdt_ms_api_erp_thirdparties

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_thirdparties`

## Descripción / propósito

**Servicio de terceros** del ERP y **fuente de verdad del `thirdparties_db`** (unas 188 migraciones). Responsabilidades:

- **CRUD de terceros**: personas naturales y jurídicas (agregado `ThirdParty` con `NaturalPerson` y `LegalPerson`), direcciones, contactos, cuentas bancarias, documentos, actividades económicas (CIIU), categorías, responsabilidades fiscales y fórmulas de liquidación.
- **Datos de cumplimiento / KYC**: PEP, accionistas, representantes legales, junta directiva, operaciones internacionales, origen de fondos y clasificación FATCA/CRS.
- **Importación masiva** desde Excel (pipeline Strategy + Factory, procesamiento asíncrono, estado de validación en Redis, notificación por WebSocket).
- **Validación externa**: listas restrictivas SARLAFT (con webhook de resultados por lote), RUES/RNEC e **Inspektor**.
- **Motor de responsabilidades fiscales**: `/fiscal-responsibilities/suggest`.
- **Numeración de documento soporte** (CXP_HU0011).
- **Selectores** para la app principal y para los portales de clientes y de pagos.

Para autenticar, lee de forma remota el dominio de usuarios (`micro_users_pgsql`, tokens Sanctum centralizados). **No** es dueño de los usuarios.

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 |
| Laravel | ^11.9 (lock v11.53.1) |
| fiduprevisora/core | dev-develop |
| Sanctum / Reverb | v4.1.1 / v1.7.0 |
| aws/aws-sdk-php + flysystem S3 | 3.388.8 / ^3.29 |
| keepsuit/laravel-opentelemetry | 2.0.0 |
| square1/laravel-idempotency, sqs-fifo-queue, predis | sí |
| maatwebsite/excel, phpspreadsheet | sí |
| Dev | PHPUnit 11, paratest, Larastan, Pint, Boost, Telescope, PHPCS |

## Arquitectura y estructura de carpetas

Capas: **Controller → Service → Model**, con Strategy/Factory en la importación masiva (ver `ARCHITECTURE.md`).

```
app/
├── Http/Controllers/Api/V1/   ThirdPartyController, ThirdPartyMassImportController,
│                              FiscalResponsibilityController, InspektorController,
│                              SettlementFormulaAssociationController, SupportDocumentNumbering, Common
├── ExternalModules/           AccountingModule, AccountsPayable, BillingCollection, ClientModule,
│                              PortalPaymentModule, SarlaftModule, TaxModule, WatchlistModule
├── Jobs/ThirdParty/           ProcessThirdPartyMassImportJob, ProcessExternalValidationBatchJob,
│                              ProcessBulkThirdPartyJob
├── Events/ Exports/ Imports/ Mail/ Models/ Services/ Enums/ Traits/ ...
database/migrations (≈188), seeders (ocupaciones, CIIU, etc.)
openspec/  specs: third-party-settlement-formulas; changes: login-token-change-driver, portal-payment-bulk-store
```

## Módulos / endpoints principales

Prefijo `api/third-parties` (en local `http://localhost:82/third-parties/api/third-parties/...`). Todo bajo `auth:sanctum` + `portal.token-boundary`, salvo los portales.

| Grupo | Rutas |
|---|---|
| Selectores | `GET select-tables` |
| Terceros | `third-parties`, `third-party-show/{id}`, `third-party-store`, `third-party-update/{id}`, `export`, `files`, `third-parties/valid-exists`, `valid-exists-third`, `third-party/{thirdParty}/status`, `apiResource third-party` |
| Direcciones | `POST addresses/check-exists` |
| Importación masiva | `mass-import/download-template`, `validate`, `valid-third-parties`, `store`, `sarlaft-webhook`, `status-findings-validation/{key}` y `/report` |
| Fiscal | `POST fiscal-responsibilities/suggest` |
| Fórmulas de liquidación | `POST settlement-formulas/third-parties` (lo invoca el módulo de impuestos) |
| Inspektor | `POST inspektor/validate` |
| Documento soporte | `support-document-numbering/third-parties` (index, show, update, toggle-status) |
| Portal | `portal/select-tables` (`portal.access`) |
| Portal de pagos | `portal-payment/select-tables`, `third-party-store`, `bulk-store`, `bulk-store/results/{batchKey}` |

Canales de broadcast (`routes/channels.php`): `App.Models.User.{id}`, `thirdparty.mass-import.{userId}`, `thirdparty.bulk-store.{userId}`.

Tareas programadas: `app:update-user-statuses` (diaria), `app:role-expiration` (diaria), `app:check-password-expiration` (diaria).

## Dependencias e integraciones

- **PostgreSQL**: `pgsql` → `thirdparties_db` (puerto local 5454), `core_pgsql` (BD principal), `micro_users_pgsql` (autenticación) y `micro_logs_pgsql`.
- **Redis**: estado de validación de la importación masiva, caché de selectores y de tokens.
- **Reverb** (`rdt_ms_api_erp_broadcasting`): progreso de la importación masiva y de los bulk-store.
- **Colas**: jobs de importación, validación externa y bulk-store. Worker `queue:work` en producción; SQS FIFO disponible.
- **AWS S3**: archivos y documentos de terceros (`AWS_ENDPOINT` permite usar MinIO en local).
- **Microservicios** (vía `ExternalModule`): sarlaft, watchlist, accounting, accounts-payable, billing-collect, clients, tax-service, portal-payment.
- **Externos**: Inspektor (`INSPEKTOR_URL`, `INSPEKTOR_USER`, `INSPEKTOR_PASSWORD`) y RUES/RNEC.
- **OpenTelemetry** y OpenAPI del core (`CORE_OPENAPI_*`), auditoría de BD del core (`CORE_DATABASE_AUDIT_*`).

## Cómo ejecutarlo localmente

`vendor/` es un volumen Docker con nombre (`thirdparties_vendor`). Todo se ejecuta dentro del contenedor del orquestador:

```bash
cd ../rdt_ms_api_erp_core && ./startup.v2.sh       # 'thirdparties' es obligatorio
docker exec -it fidu-thirdparties php artisan migrate
docker exec -it fidu-thirdparties php artisan test --parallel --processes=4
docker exec -it fidu-thirdparties ./vendor/bin/pint --test
```

Scripts auxiliares: `core-dev.sh`, `core-tools.sh` (sincronización con el paquete core) y `composer-dev.json`.

## Variables de entorno (solo nombres)

`APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `API_EXTERNAL`, `API_ENVIRONMENT`, `CORS_ALLOWED_ORIGINS`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `APP_MAINTENANCE_DRIVER`, `APP_MAINTENANCE_STORE`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_CORE_PGSQL_*` (HOST, PORT, DATABASE, USERNAME, PASSWORD), `DB_MICRO_USERS_PGSQL_*` (CONNECTION, HOST, PORT, DATABASE, USERNAME, PASSWORD), `LDAP_*`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `CACHE_STORE`, `CACHE_PREFIX`, `TOKEN_CACHE_*`, `MEMCACHED_HOST`, `REDIS_CLIENT`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `MAIL_*`, `AWS_ENDPOINT`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, `AWS_USE_PATH_STYLE_ENDPOINT`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `REVERB_HOST`, `REVERB_PORT`, `REVERB_SCHEME`, `BROADCAST_CONNECTION`, `REVERB_APP_ID`, `REVERB_APP_KEY`, `REVERB_APP_SECRET`, `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_PROTOCOL`, `OTEL_TRACES_SAMPLER`, `CORE_OPENAPI_ENABLED`, `CORE_OPENAPI_SERVICE_PATH`, `CORE_OPENAPI_ROUTE_MIDDLEWARES`, `SERVICES_INTERNAL_TOKEN`, `SERVICE_TOKEN_TTL`, `CORE_DATABASE_AUDIT_ENABLED`, `CORE_DATABASE_AUDIT_AUTO_MIDDLEWARE`, `CORE_DATABASE_AUDIT_ACTIVITY_MODEL`. En `config/inspektor.php`: `INSPEKTOR_URL`, `INSPEKTOR_USER`, `INSPEKTOR_PASSWORD`.

## Pruebas

PHPUnit 11 con paratest (unos 39 archivos de test) sobre SQLite en memoria (`micro_users_pgsql` también se apunta a SQLite). En el entorno de pruebas se desactivan activity log, Telescope y OTEL. `phpunit.xml` excluye muchos directorios de la cobertura. CI (`azure-pipeline.yaml`, develop/qa/uat): `composer install` → `php artisan test --parallel --processes=4` con Cobertura → **Grype** (`--fail-on critical`) → **Bearer** SAST. Pint no corre en CI.

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `2206f657` (2026-08-25) Merged PR 216071: feature: FATCA/CRS classification of a customer structure table |
| Remoto | `https://dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_thirdparties` |

## Notas

- Los commits deben seguir Conventional Commits (hook `.githooks/commit-msg`).
- Archivos sobrantes: `azure-pipeline copy.yaml`, `azure-pipeline copy 2.yaml`, `docker-dev/php-fpm/Dockerfile copy`.
- `.mcp.json` configura el servidor MCP `laravel-boost`.
- Arrastra scaffolding de autenticación/usuarios heredado de la plantilla (servicios de usuario y comandos de estado). Es infraestructura, no dominio propio.
