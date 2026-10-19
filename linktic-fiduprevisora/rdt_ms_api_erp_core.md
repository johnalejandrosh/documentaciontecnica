# rdt_ms_api_erp_core

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_core`

## Descripción / propósito

Es el **repositorio orquestador y dueño del esquema** del ERP Fiduprevisora (sistema de gestión fiduciaria colombiano). Según su `AGENTS.md`, cumple cuatro funciones:

1. **Orquestador Docker Compose**: levanta toda la plataforma local (más de 33 microservicios, más de 23 instancias PostgreSQL, Redis, MinIO, Nginx, Typesense, Reverb, observabilidad).
2. **Balanceador Nginx**: enruta el tráfico del puerto `82` al contenedor PHP-FPM que corresponde según el prefijo de URL (`/<url>/api/<url>`).
3. **Dueño del esquema de base de datos**: unos 1.246 archivos de migración que cubren todos los dominios.
4. **Código compartido**: traits, middleware, helpers y modelos que se sincronizan con el paquete `fiduprevisora/core`.

`routes/app.php` está vacío **a propósito**: el core no expone endpoints de negocio. Solo tiene la ruta `/api/core` (vista de bienvenida).

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 (imagen PHP-FPM 8.2) |
| Laravel | ^11.9 (lock: v11.53.1) |
| fiduprevisora/core | dev-develop |
| Sanctum | v4.0.8 |
| spatie/laravel-permission / activitylog / query-builder / pdf | 6.x / 4.x / 6.x / 1.x |
| maatwebsite/excel, phpspreadsheet, dompdf, google2fa, simple-qrcode | sí |
| Dev | Pint, Sail, Telescope, PHPUnit 11, symplify/monorepo-builder |
| Infra local | PostgreSQL 15-alpine, Redis alpine, MinIO, Typesense 26.0, OTel Collector + Jaeger |

## Arquitectura y estructura de carpetas

```
app/            Console, Contracts, Enums, Exceptions, Exports, Helpers, Http, Imports, Mail,
                Models, Observers, Pagination, Providers, Rules, Services, Traits
database/       migrations (≈1.246), seeders (catálogos: estados, tipos de documento,
                países, departamentos/ciudades, ocupaciones, bancos, etc.), factories
db/             init.sql, init-tax-service.sql, my.cnf
docker/         imagen productiva (php-fpm con crontab + overlay laravel-worker, nginx)
docker-dev/     imagen de desarrollo (PHP-FPM 8.2 con PCOV, Composer fijado)
infra/          CloudFormation AWS (master, service, LoadBalancerV2, Security_Groups, parameters)
nginx/          nginx.conf, sites/default.conf (autogenerado), ssl
scripts/        clone_microservices.sh, update_microservices.sh, register_microservice.sh, lib, tests
services.conf   fuente única de verdad de microservicios (repo:url:contenedor:perfil:prioridad)
docker-compose.v2.yml   compose activo (anchors YAML, healthchecks TCP, .env.docker)
docker-compose.yml      versión original (solo rollback)
```

## Módulos / endpoints principales

- `GET /api/core` → vista `welcome`.
- `routes/app.php`, `app_v2.php`, `app_v3.php`: vacíos.
- **Tareas programadas** (`routes/console.php`): `app:update-user-statuses` (diaria), `app:reset-authentication-method-failed-attempts` (cada hora), `app:role-expiration` (diaria), `app:check-password-expiration` (diaria). En el stack local corren en el contenedor `scheduler` (`php artisan schedule:work`).

**Enrutamiento Nginx** (`nginx/sites/default.conf`, generado): por ejemplo `/users/api/users → rdt-users:9000`, `/third-parties/api/third-parties → rdt-thirdparties:9000`, `/reporter/api/reporter → rdt-reporter:9000`.

`services.conf` define como **obligatorios** `core`, `users`, `security`, `thirdparties` y `broadcasting`, y como **opcionales** unos 30 más: clients, assets, fixed_assets, treasury, accounting, business_trust, investment_portfolio, portfolio, fic, financial_obligations, derivative_contracting, budget, accounts_payable, billing_collection, commission_settlement, sarlaft, normative, watchlist_integration, seizure, bank_reconciliation, audit_log, accountability, agenda_notificaciones, core_portals, portal_fics, portal_payment, tax_service, orion_integration, fidugestor_integration, ims_integration y reporter.

## Dependencias e integraciones

- **PostgreSQL**: BD principal `fidudb` (5432) más una BD por microservicio (puertos 5433 a 5455; por ejemplo `users_db` 5452, `thirdparties_db` 5454, `logs_db` 5436, `fidudb_test` 5455). Conexiones en `config/database.php`: `pgsql`, `micro_users_pgsql`, `micro_third_parties_pgsql`, `micro_logs_pgsql`.
- **Redis**: caché y colas.
- **MinIO**: almacenamiento compatible con S3 en local. En AWS se usa S3.
- **Reverb** (`rdt_ms_api_erp_broadcasting`): WebSockets en el puerto 8080.
- **Typesense** (8108): búsqueda full-text (la usa `users`).
- **OpenTelemetry + Jaeger** (4317 / 16686): trazas distribuidas (opcional con `--with-observability`).
- **DynamoDB local** (opcional con `--with-dynamodb`).
- **AWS** (despliegue): ECS (TaskDefinition/Service), ALB TargetGroup, Security Groups, CloudWatch Logs, mediante las plantillas CloudFormation de `infra/`.
- **Paquete core**: se monta en `/var/www/core-package` en cada contenedor. `sync-models.sh` copia `app/Models` hacia `rdt_ms_api_erp_core_package/src/Models`.

## Cómo ejecutarlo localmente

Requisitos: los repos hermanos clonados en el mismo directorio padre (`../rdt_ms_api_erp_*`) y `AZURE_TOKEN` para que Composer descargue `fiduprevisora/core`.

```bash
# 1. Clonar/actualizar los microservicios definidos en services.conf (SSH a Azure DevOps)
./scripts/clone_microservices.sh
./scripts/update_microservices.sh

# 2. Regenerar la configuración Nginx a partir de los repos hermanos
./generate_nginx_config.v2.sh            # CORS_ORIGIN=https://... para sobrescribir '*'

# 3. Levantar el stack (detecta repos y arma los --profile)
./startup.v2.sh                          # menú interactivo
./startup.v2.sh all
./startup.v2.sh accounting fic
./startup.v2.sh --with-observability --with-dynamodb all

# Manual
docker compose -f docker-compose.v2.yml --profile users --profile accounting up -d

# Apagar
./teardown.v2.sh
```

El stack queda disponible en `http://localhost:82`. Dentro del contenedor `fidu-core` se usan `php artisan migrate`, `php artisan db:seed` y `./vendor/bin/pint`. Para registrar un microservicio nuevo, la guía es `docs/REGISTRAR_MICROSERVICIO.md` (hay que tocar `services.conf`, `docker-compose.v2.yml` y, si aplica, crear la migración).

## Variables de entorno (solo nombres)

- `.env.example`: `APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `SEED_USERS_PASSWORD`, `APP_MAINTENANCE_DRIVER`, `APP_MAINTENANCE_STORE`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_MICRO_USERS_PGSQL_*` (CONNECTION, HOST, PORT, DATABASE, USERNAME, PASSWORD), `DB_THIRDPARTIES_PGSQL_*`, `LDAP_ENABLE`, `LDAP_HOST`, `LDAP_PORT`, `LDAP_BASE_DN`, `LDAP_BASE_DOMAIN`, `BROADCAST_CONNECTION`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `CACHE_STORE`, `CACHE_PREFIX`, `MEMCACHED_HOST`, `REDIS_CLIENT`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `MAIL_*`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, `AWS_USE_PATH_STYLE_ENDPOINT`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `AZURE_TOKEN`.
- `.env.docker` (compartido por el compose v2): `APP_ENV`, `APP_DEBUG`, `REDIS_HOST`, `REDIS_PORT`, `AZURE_TOKEN`, `COMPOSER_PROCESS_TIMEOUT`, `COMPOSER_MEMORY_LIMIT`.
- Compose: `DB_<SERVICIO>_DATABASE/USERNAME/PASSWORD` por cada Postgres, y `REVERB_APP_ID`, `REVERB_APP_KEY`, `REVERB_APP_SECRET`, `REVERB_ALLOWED_ORIGINS`.

## Pruebas

```bash
php artisan test                         # PHPUnit 11 (unos 46 tests)
php artisan test --testsuite=Feature|Unit
php artisan test --parallel
./run-tests.sh                           # runner multi-servicio (contenedor test-runner)
./vendor/bin/pint --test
```

Las pruebas usan `RefreshDatabase` con auto-seed y SQLite en memoria (ver `phpunit.xml`).

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `4a5cf3da` (2026-08-26) Merged PR 216620: feat: make has_cash_flow nullable in business_trust_treasuries table - HU0010 HU0057 |
| Remoto | `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_core` |
| CI | `azure-pipeline.yaml` (trigger `develop`, `qa`): `ValidateAndTest` → `SyncMigrations` (`sync-migrations.sh` copia migraciones a repos dependientes como `rdt_ms_api_erp_business_trust`) → build base PHP |

Actividad reciente: migraciones de tesorería, registros de ingresos (comisión/IVA), sucursales bancarias y parámetros de integración de tesorería.

## Notas

- La imagen productiva corre migraciones y seeders al construirse. La de desarrollo no: ahí se ejecutan a mano.
- Hay que respetar el patrón de caché de capas en los Dockerfile (copiar `composer.json`/`composer.lock` antes que el código) y definir `COMPOSER_PROCESS_TIMEOUT=900`.
- PHP-FPM: `memory_limit 2048M`, `max_execution_time 900`, `upload_max_filesize 500M`.
- Hay archivos sobrantes en el repo: `docker-compose copy.yml` y los scripts v1 (`startup.sh`, `generate_nginx_config.sh`). Se prefieren las versiones v2.
- `README.md` es la plantilla por defecto de Azure DevOps. La documentación real está en `AGENTS.md`, `docs/` y `scripts/README.md`.
