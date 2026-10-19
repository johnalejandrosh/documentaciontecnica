# rdt_ms_api_erp_security

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_security`

## Descripción / propósito

**Servicio de autenticación y seguridad** del ERP. No tiene dominio de negocio propio ni migraciones reales: es un **lector remoto** del `users_db`, cuyo dueño es `rdt_ms_api_erp_users`. Responsabilidades:

- **Login** local (email/contraseña) y **LDAP / Azure AD**, ambos con throttle `5,1`.
- **Ciclo de vida de tokens**: emisión de access y refresh token, rotación, logout y límite de sesiones concurrentes.
- **Protección de cuenta**: bloqueo por intentos fallidos, expiración de contraseña y control por estado del usuario.
- **Dashboards de seguridad** (solo lectura y exportación): usuarios conectados, estado de usuarios, intentos fallidos, tiempos de conexión, horarios de ingreso y control de versiones de módulos.
- **Sincronización de usuarios con Azure AD** (`app:sync-azure-users`, cada hora).

El 2FA **no** es un control requerido por decisión de negocio (ADR en `openspec/changes/security-audit-round-2`). El scaffolding de google2fa está inactivo.

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 |
| Laravel | ^12.0 (lock v12.64.0) |
| fiduprevisora/core | dev-chore/allow-illuminate-12 |
| Sanctum | v4.3.2 |
| aws/aws-sdk-php | 3.388.8 |
| keepsuit/laravel-opentelemetry | 2.2.4 |
| shiftonelabs/laravel-sqs-fifo-queue, predis | ^3.0 / ^3.2 |
| Pruebas | Pest v3.8.7 + PHPUnit 11 |
| Calidad | Larastan (baseline), Rector, Pint, PHPCS, Bearer, Sonar |

## Arquitectura y estructura de carpetas

Capas: **Controller → Service → Model** (ver `ARCHITECTURE.md` y `CLAUDE.md`).

```
app/  Console, Exceptions, Exports, Helpers, Http (Controllers, Services: LoginService,
      LDAPService, ...), Jobs (LogAuthenticationJob, LogFailedLoginAttemptJob, SyncAzureUsersJob),
      Models, Observers, Providers, Rules, Services
routes/app.php      rutas V1
routes/app_v2.php   rutas V2 montadas bajo /v2 (user-connections, entry-times/list)
tests/database/migrations   esquema solo para pruebas (réplica de las tablas de users_db que se usan)
openspec/  specs: token-lifecycle, token-cache, secret-hygiene, secure-azure-transport,
           azure-user-status-sync, ldap-user-provisioning, login-schedule-report, ...
infra/     CloudFormation (ECS, ALB, SG)
```

## Módulos / endpoints principales

Prefijo `api/security`, o `<API_ENVIRONMENT>/api/security`.

| Método | Ruta | Notas |
|---|---|---|
| POST | `login` | throttle 5/min |
| POST | `login/ldap` | Azure AD / LDAP, throttle 5/min |
| POST | `refresh` | rotación de token |
| POST | `logout` | sanctum |
| GET | `select-tables` | selectores |
| GET | `modules-app`, `modules-app/export` | `module-security-module-version-control*` |
| GET | `users-status`, `/list`, `/export` | `module-security-user-status*` |
| GET | `failed-login-attempts`, `/export` | `module-security-failed-login-attempts*` |
| GET | `connected-users/count`, `/list`, `/export` | `module-security-connected-users*` |
| GET | `user-connections`, `/export` | `module-security-user-connection-time*` |
| GET | `entry-times`, `/list`, `/export` | `module-security-user-login-schedule*` |
| GET | `v2/user-connections`, `v2/entry-times/list` | V2 independiente de V1 |

Las versiones V1 y V2 se mantienen **independientes a propósito**: V1 queda congelada como respaldo para revertir si V2 falla en producción.

Tareas programadas: `app:check-password-expiration` (diaria) y `app:sync-azure-users` (cada hora). La inactivación por días sin login está retirada porque Azure AD gobierna el ciclo de vida.

## Dependencias e integraciones

- **PostgreSQL**: `micro_users_pgsql` (lectura remota del `users_db`, incluida `personal_access_tokens`) y `micro_logs_pgsql` (logs de autenticación). `pgsql` por defecto.
- **Redis**: caché de tokens (`TOKEN_CACHE_*`) y presencia de usuarios conectados.
- **Colas**: jobs de log de autenticación e intentos fallidos. En producción, worker `queue:work` en el overlay `docker/php-fpm/overlay/laravel-worker`. SQS FIFO disponible.
- **Azure AD / LDAP**: `LDAPService` y `SyncAzureUsersJob` (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, etc.).
- **OpenTelemetry**: exportador OTLP (`OTEL_*`).
- **AWS**: S3 para exportaciones, ECS para el despliegue, EFS para storage (`entrypoint.sh` prepara `storage/` montado en EFS).
- **Microservicios**: `users` (dueño del esquema y de los tokens).

## Cómo ejecutarlo localmente

Igual que en `users`, no se corre PHP en el host porque el `vendor/` apunta a `/var/www/core-package`. Se usa el orquestador:

```bash
cd ../rdt_ms_api_erp_core && ./startup.v2.sh       # 'security' es obligatorio
docker exec -it fidu-security php artisan test
docker exec -it fidu-security ./vendor/bin/pint --test
# cobertura
docker exec -it fidu-security php -d pcov.enabled=1 -d pcov.directory=app ./vendor/bin/pest --coverage-clover coverage/clover.xml
```

Existe un `docker-compose.yml` propio (nginx 82 + php-fpm + postgres 5432) heredado de la plantilla.

## Variables de entorno (solo nombres)

`APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `API_ENVIRONMENT`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `APP_MAINTENANCE_DRIVER`, `APP_MAINTENANCE_STORE`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_SSLMODE`, `DB_MICRO_USERS_PGSQL_SSLMODE`, `DB_LOGS_PGSQL_SSLMODE`, `CORS_ALLOWED_ORIGINS`, `TRUSTED_PROXIES`, `LDAP_ENABLE`, `LDAP_HOST`, `LDAP_PORT`, `LDAP_BASE_DN`, `LDAP_BASE_DOMAIN`, `BROADCAST_CONNECTION`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `MEMCACHED_HOST`, `CACHE_STORE`, `CACHE_DRIVER`, `REDIS_CLIENT`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `REDIS_DB`, `CACHE_PREFIX`, `TOKEN_CACHE_ENABLED`, `TOKEN_CACHE_STORE`, `TOKEN_CACHE_TTL`, `TOKEN_CACHE_LAST_USED_INTERVAL`, `TOKEN_CACHE_PREFIX`, `MAIL_*`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, `AWS_USE_PATH_STYLE_ENDPOINT`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `AZURE_TOKEN`, `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_PROTOCOL`, `OTEL_TRACES_SAMPLER`, `SERVICES_INTERNAL_TOKEN`, `SERVICE_TOKEN_TTL`, `ACTIVITY_LOGGER_DB_CONNECTION`, `ACTIVITY_LOGGER_TABLE_NAME`. En `config/`: `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_REDIRECT_URI`.

## Pruebas

Pest + PHPUnit 11 (unos 28 archivos de test) sobre SQLite en memoria. Las conexiones `micro_users_pgsql` y `micro_logs_pgsql` se apuntan al mismo PDO SQLite, y se simulan (fake) `Queue` y `Redis`. Hay que mantener `APP_ENV=local` en `phpunit.xml`, porque con `testing` el `PermissionMiddleware` del core omite las validaciones de permisos. CI (`azure-pipeline.yaml`, develop/qa): `ValidateAndTest` con los exportadores OTEL desactivados.

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `b6af5b4` (2026-08-19) Merged PR 214334: refactor(security): SecurityHeaders centralizado en el paquete core |
| Remoto | `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_security` |

## Notas

- Los commits deben seguir Conventional Commits (hook `.githooks/commit-msg`).
- Varios dashboards están duplicados en `users` y están en proceso de migrar a este servicio.
- Hay mucha documentación OpenSpec (`openspec/changes`: dead-code-purge, security-audit-round-2, upgrade-laravel-12, fix-connected-users-presence, etc.).
- Hay archivos sobrantes: `docker-dev/php-fpm/Dockerfile copy`. `README.md` es la plantilla por defecto de Azure DevOps.
