# rdt_ms_api_erp_core_package (`fiduprevisora/core`)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_core_package`

## Descripción / propósito

Paquete Composer compartido (`fiduprevisora/core`, "Core for Fidu application") que consumen **todos** los microservicios Laravel del ERP Fiduprevisora. Concentra la infraestructura transversal para que cada micro no la reimplemente:

- **Comunicación entre microservicios**: `BaseExternalModule` / `QueryableExternalModule` (cliente HTTP con resolución de URL por módulo, autenticación servicio-a-servicio, propagación del usuario original, caché de GET, pool concurrente con fibers, respuestas binarias). El registro de módulos (`ExternalModule::_MODULES`) lista unos 30 destinos: `security`, `users`, `third-parties`, `treasuries`, `accounting`, `business-trust`, `fics`, `reporter`, `tax-service`, `sarlaft`, `portal-*`, etc.
- **Gateway** (`src/Gateway`): despacho de acciones con estrategias, políticas y adaptadores de API externa.
- **Autenticación de servicio**: `ServiceTokenGuard`, `ServiceTokenManager`, `ServiceIdentity`, `UserResolver` / `SystemUserResolver`.
- **Middlewares comunes**: `AuditLogMiddleware`, `CheckUserSessionMiddleware`, `PermissionMiddleware`, `PortalAccessMiddleware`, `PortalTokenBoundaryMiddleware`, `PortalTransactionalAccessMiddleware`, `SecurityHeaders`, `TrustProxies`, `DatabaseAuditContextMiddleware`.
- **Auditoría**: activity log, auditoría de base de datos con tabla *outbox* y comando de vaciado, `RelationGuard` (con respaldo en DynamoDB).
- **Caché**: `ResourceCacheService` (invalidación de selectores en Redis con `SCAN`, por convención de nombre de tabla), `TokenCacheTrait` (caché de tokens Sanctum).
- **Utilidades**: `BaseUtilsController` V1/V2 (endpoint `select-tables`), traits (`ResponseApiTrait`, `ResourcesHandlerTrait*`, `FileExportableTrait`, `DefaultLogs`, `InsensitiveSearch`…), helpers (`Pick`, `HolidaysHelper`, `AuditLogsHelper`), servicios S3/DynamoDB, presencia de usuarios en Redis, OpenAPI (Scramble), mail asíncrono por SES.
- **Modelos compartidos** (`src/Models`, unos 396 archivos): Accounting, Asset, BusinessTrust, Client, Fics, Holiday, Main, Notifications, Reports, ReportTemplate, ThirdParty, Treasury, User.
- **Revisión de PR con LLM** (`ReviewPrCommand`, `src/Services/CodeReview`) integrada con Azure DevOps.

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 |
| illuminate/* | ^10.0 \|\| ^11.0 en `develop` (los servicios en Laravel 12 usan la rama `chore/allow-illuminate-12`) |
| spatie/laravel-package-tools | ^1.16 |
| dedoc/scramble | ^0.13.14 (OpenAPI) |
| async-aws/dynamo-db | ^3.10 |
| symfony/amazon-mailer | ^6.4 \|\| ^7.0 |
| Pruebas | Pest (+ arch, laravel), Orchestra Testbench, Larastan/PHPStan |

## Arquitectura y estructura de carpetas

```
src/
├── Auth/              # guard y tokens de servicio
├── Class/             # CoreModel, CoreUser, relaciones con activity log en pivotes
├── Commands/          # core:sync, core:sync-restore, publicar modelos, auditoría, review-pr
├── config/            # activity-log, audit-log, cache-invalidation, database-audit, openapi,
│                      # presence, relation-guard(-dynamo), service-auth, token-cache, code-review
├── Contracts/ Enums/ Facades/ Helpers/ Macros/ Rules/ Support/ Traits/
├── Gateway/           # Application / Domain / Infrastructure
├── Http/              # Controllers base, ExternalModule, Middleware, Resources, Services (S3, Dynamo)
├── Jobs/              # auditoría, exportación de archivos, invalidación de caché, relaciones
├── Models/            # modelos de dominio compartidos
├── Observers/ Registry/ RelationGuard/ Services/ Testing/ stubs/
└── *ServiceProvider.php
docs/                  # guías en español (ver Notas)
tests/                 # Pest (unos 60 archivos)
```

## Módulos / endpoints principales

No expone API propia (no tiene `routes/`). Aporta a los micros el controlador base de selectores (`select-tables`), la UI/JSON de OpenAPI opcional (`CORE_OPENAPI_*`) y comandos Artisan (`core:sync`, auditoría, publicación de modelos, review de PR).

## Dependencias e integraciones

- **Redis**: caché de recursos/selectores, caché de tokens, presencia de usuarios.
- **AWS**: S3 (`S3Service`), DynamoDB (RelationGuard, `DynamoService`), SES (mail asíncrono).
- **Azure DevOps**: el pipeline sincroniza los modelos hacia microservicios dependientes; la revisión de PR usa la API de ADO y un LLM configurable.
- **Otros microservicios**: vía `ExternalModule` (HTTP con token interno de servicio).

## Cómo ejecutarlo localmente

No corre solo: se monta como *path repository* en cada micro (volumen `../rdt_ms_api_erp_core_package/ → /var/www/core-package/` en `rdt_ms_api_erp_core/docker-compose.v2.yml`). Para trabajar el paquete:

```bash
composer install
composer test          # Pest
composer analyse       # PHPStan
composer format        # Pint
```

`core-dev.sh` (para uso desde un micro) actualiza el paquete con `git pull` y sincroniza modelos con `php artisan core:sync core-to-repo` / `repo-to-core`.

## Variables de entorno (solo nombres)

`API_ENVIRONMENT`, `CORE_PACKAGE_PATH`, `REPO_MODELS_PATH`, `INTERNAL_SERVICE_TOKEN`, `SERVICES_INTERNAL_TOKEN`, `SERVICE_TOKEN_TTL`, `PORTAL_USER_MODEL`, `TOKEN_CACHE_ENABLED`, `TOKEN_CACHE_STORE`, `TOKEN_CACHE_TTL`, `TOKEN_CACHE_PREFIX`, `TOKEN_CACHE_LAST_USED_INTERVAL`, `PRESENCE_ENABLED`, `PRESENCE_KEY`, `PRESENCE_GUARD_PREFIX`, `PRESENCE_REDIS_CONNECTION`, `PRESENCE_THROTTLE_SECONDS`, `CORE_DATABASE_AUDIT_ENABLED`, `CORE_DATABASE_AUDIT_AUTO_MIDDLEWARE`, `CORE_DATABASE_AUDIT_ACTIVITY_MODEL`, `CORE_DATABASE_AUDIT_FLUSH_ENABLED`, `CORE_DATABASE_AUDIT_OUTBOX_TABLE`, `CORE_OPENAPI_ENABLED`, `CORE_OPENAPI_UI_ENABLED`, `CORE_OPENAPI_UI_PATH`, `CORE_OPENAPI_DOCUMENT_PATH`, `CORE_OPENAPI_SERVICE_PATH`, `CORE_OPENAPI_ROUTE_MIDDLEWARES`, `CORE_OPENAPI_SCRAMBLE_MIDDLEWARES`, `RELATION_GUARD_ENABLED`, `RELATION_GUARD_DYNAMO_*` (TABLE, GSI, REGION, ENDPOINT, ACCESS_KEY_ID, ACCESS_KEY_SECRET, SESSION_TOKEN), `RELATION_GUARD_OUTBOX_*`, `RELATION_GUARD_DRAIN_OUTBOX_*`, `RELATION_GUARD_REPLAY_OUTBOX_*`, `RELATION_GUARD_DATABASE_CAPTURE_BATCH_SIZE`, `CODE_REVIEW_ADO_ORG`, `CODE_REVIEW_ADO_PROJECT`, `CODE_REVIEW_ADO_PAT`, `CODE_REVIEW_REPO_ID`, `CODE_REVIEW_LLM_URL`, `CODE_REVIEW_LLM_KEY`, `CODE_REVIEW_LLM_MODEL`, `CODE_REVIEW_LLM_TIMEOUT`, `CODE_REVIEW_MAX_CONTEXT`.

## Pruebas

Pest + Orchestra Testbench (`phpunit.xml.dist`), unos 60 archivos en `tests/`: middlewares, tokens de servicio, pool de módulos externos, auditoría, RelationGuard, presencia, security headers, tests de arquitectura (`ArchTest.php`). PHPStan con baseline (`phpstan-baseline.neon`).

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `7448fee` (2026-08-24) Merged PR 216051: fix(logs): evita modificación indirecta de Collection en record_id |
| Remoto | `https://dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_core_package` |
| CI | `azure-pipeline.yaml` (trigger `develop`): etapas `Setup` y `SyncChangesToMicroservices` (propaga cambios de modelos a business_trust, accounting y treasury) |

Commits recientes: auditoría con `record_id`, validaciones SARLAFT para portal FICS, outbox multi-microservicio, middleware de security headers.

## Notas

- Guías internas en `docs/`: `base-external-module-guia.md`, `queryable-external-module-guia.md`, `external-module-pool-guia.md`, `cache-invalidation-guide.md`, `database-audit-guia-configuracion.md`, `relation-guard-guia-operativa.md`, `portal-access-boundary.md`, `security-headers.md`, `pick-helper-guia.md`, `builder-macros-guia.md`, `sdd-proposal.md` (invalidación unificada de caché de selectores).
- Los micros referencian el paquete desde el repositorio de Azure DevOps (`repositories` en sus `composer.json`), así que para construir imágenes se necesita `AZURE_TOKEN`.
- Los micros en Laravel 12 (`users`, `security`) apuntan a `dev-chore/allow-illuminate-12`; el resto usa `dev-develop`.
