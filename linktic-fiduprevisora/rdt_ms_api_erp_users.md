# rdt_ms_api_erp_users

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_users`

## Descripción / propósito

Es el **servicio de identidad y autenticación** del ERP y la **fuente de verdad del `users_db`**: es dueño del esquema (unas 100 migraciones), de los seeders y de las factories. Responsabilidades:

- CRUD de identidad: usuarios, roles, permisos, parámetros globales (`UserParameter`), niveles de autorización y opciones de inactivación de rol.
- Autenticación local (email/contraseña) y **LDAP / Azure AD**, con refresh y logout.
- 2FA (TOTP con google2fa + QR) y métodos por email/SMS.
- Recuperación de contraseña con preguntas de seguridad y código por email.
- Perfil de autoservicio: datos, foto, contraseña y firma digital en S3.
- Reportes de usuario asíncronos (PDF/Excel, patrón Factory).
- Búsqueda full-text de usuarios y terceros en **Typesense** vía Laravel Scout.
- **Tokens Sanctum centralizados**: la tabla `personal_access_tokens` vive aquí y todos los micros autentican contra ella.
- Dashboards de seguridad (intentos fallidos, usuarios conectados, tiempos de conexión, etc.) **en migración** hacia `rdt_ms_api_erp_security` (están marcados `//Pasar luego al repo de security - ELIMINAR`).

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 |
| Laravel | ^12.0 (lock v12.64.0) |
| fiduprevisora/core | dev-chore/allow-illuminate-12 |
| Sanctum | v4.1.1 |
| Laravel Scout + typesense-php | ^10.15 / v5.1.0 |
| aws/aws-sdk-php + flysystem-aws-s3-v3 | 3.388 / ^3.0 |
| shiftonelabs/laravel-sqs-fifo-queue | ^3.0 |
| square1/laravel-idempotency | ^12.2 |
| predis | ^3.2 |
| spatie permission / activitylog / query-builder | 6.20 / 4.x / 6.x |
| Dev | PHPUnit 11, paratest, Larastan, Rector, Pint, Boost, Telescope, PHPCS |

## Arquitectura y estructura de carpetas

Capas: **controlador delgado → Service → Model** (sin Strategy). Ver `ARCHITECTURE.md` y `CLAUDE.md` del repo.

```
app/
├── Console/Commands/   # ValidateTemporaryRole, ValidateLastLogin, CheckPasswordExpiration,
│                       # DeactivateInactiveUsers, RoleExpiration, SyncAzureUsers, ...
├── Http/Controllers/Api/Vn, Http/Services/   # la mayoría de la lógica
├── Services/           # EmailService, PdfService, SmsApi, ValidationCodeService, V1/User/Report
├── Jobs/               # GenerateReportJob, LogAuthenticationJob, LogFailedLoginAttemptJob,
│                       # SyncAzureUsersJob, UpdateUserActivityJob
├── Models/ (User, Typesense/Searchable*), Enums, Exports, Imports, Mail, Observers, Rules, Traits
database/migrations (≈100), seeders, factories
openspec/  (specs: ci-test-gate, codebase-dead-code-removal, dependency-currency,
            role-portal-exclusion, versioned-secret-hygiene)
infra/     CloudFormation (ECS Service/TaskDefinition, ALB TargetGroup, SG, LogGroups, IAM Role)
```

## Módulos / endpoints principales

Prefijo `api/users`, o `<API_ENVIRONMENT>/api/users` (en local `users/api/users`). Rutas en `routes/app.php`.

| Grupo | Rutas (resumen) | Auth |
|---|---|---|
| Login | `login`, `login/ldap`, `refresh`, `logout` | públicas (con throttle), salvo logout |
| Recuperación | `recover/security-data`, `recover/password`, `check-email-code` | públicas (con throttle) |
| Interfaz | `interface-configuration/available` | pública |
| Usuarios | `fidu-users/get`, `u/{user}`, `new-user`, `update/{user}`, `delete/{user}`, `{user}/unlock`, `{user}/change-status`, `update-password/{user}`, `export` | sanctum + `module-user-management*` |
| Firma | `fidu-users/{user}/signature/presigned-url`, `activate`, `{signature}` | sanctum |
| Parámetros | `GET/PUT parameters` | `module-user-parameters*` |
| Roles | `roles` CRUD + `change-status/{role}` | `module-user-roles*` |
| Perfil | `profile`, `update`, `update-photo`, `update-password`, `authentication-method/*` (2FA, OTP, SMS/email, logs), `security-question/*` | sanctum |
| Selectores | `select-tables`, `select-tables-v2`, `resource/module-app`, `resource/role-inactive-options`, `resource/authorization-level` | sanctum |
| Permisos | `permissions` | sanctum |
| Dashboards (en migración) | `failed-login-attempts`, `connected-users`, `user-connections`, `entry-times`, `users-status`, `modules-app` (+ `export`) | `module-security-*` |
| Reportes | `user-reports/list`, `generate`, `download/{reportable}` | `module-user-reports*` |
| Configuración de interfaz | `interface-configuration/list`, `store`, `show`, `update`, `delete`, `toggle-status` | `module-interface-parametrization*` |
| Portales | `portal/u/{user}`, `portal-payment/select-tables` | `portal.access` |

Middleware global: CORS, `CheckUserSessionMiddleware` (core), `SessionTimeout`, `SecurityHeaders`, `AuditLogMiddleware` (core), `IdempotencyMiddleware` (cabecera `Idempotency-Key`), `portal.token-boundary`. Tarea programada: `app:validate-temporary-role-command` (en `routes/console.php`).

## Dependencias e integraciones

- **PostgreSQL**: `pgsql` → `users_db` (puerto local 5452) y `micro_logs_pgsql` (auditoría/logs).
- **Redis**: caché de tokens (`TOKEN_CACHE_*`), usuarios conectados (presencia), caché de selectores.
- **Typesense**: índice de usuarios y terceros (`php artisan scout:import "App\Models\Typesense\SearchableUser"`).
- **AWS**: S3 (fotos, firmas, reportes), SQS / SQS FIFO como driver de cola opcional. En el contenedor de desarrollo el worker usa `queue:work database`.
- **Azure AD / LDAP**: login y sincronización de usuarios (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`…).
- **SMS / email**: `SmsApi` y mailer.
- **Otros micros**: `security` lee el `users_db` de forma remota, y todos los micros validan tokens contra `personal_access_tokens`.
- **Paquete core**: `fiduprevisora/core` (rama para Laravel 12).

## Cómo ejecutarlo localmente

No se recomienda correr PHP/Composer en el host: el `vendor/` del host está roto porque el path repo del core apunta a una ruta del contenedor. Se usa el stack del orquestador:

```bash
cd ../rdt_ms_api_erp_core && ./startup.v2.sh        # 'users' es obligatorio
docker exec -it fidu-users php artisan migrate --seed
docker exec -it fidu-users php artisan test
docker exec -it fidu-users ./vendor/bin/pint --test
```

URL: `http://localhost:82/users/api/users/...`. Existe también un `docker-compose.yml` propio (nginx en 82 + php-fpm + postgres en 5432), pero es una alternativa heredada de la plantilla.

## Variables de entorno (solo nombres)

`APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `API_ENVIRONMENT`, `CORS_ALLOWED_ORIGINS`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `APP_MAINTENANCE_DRIVER`, `APP_MAINTENANCE_STORE`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `LDAP_ENABLE`, `LDAP_HOST`, `LDAP_PORT`, `LDAP_BASE_DN`, `LDAP_BASE_DOMAIN`, `BROADCAST_CONNECTION`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `CACHE_STORE`, `CACHE_PREFIX`, `TOKEN_CACHE_ENABLED`, `TOKEN_CACHE_STORE`, `TOKEN_CACHE_TTL`, `TOKEN_CACHE_LAST_USED_INTERVAL`, `TOKEN_CACHE_PREFIX`, `MEMCACHED_HOST`, `REDIS_CLIENT`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `MAIL_*`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, `AWS_USE_PATH_STYLE_ENDPOINT`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `CORE_OPENAPI_ENABLED`, `CORE_OPENAPI_SERVICE_PATH`, `CORE_OPENAPI_ROUTE_MIDDLEWARES`, `SERVICES_INTERNAL_TOKEN`, `SERVICE_TOKEN_TTL`, `ACTIVITY_LOGGER_DB_CONNECTION`, `ACTIVITY_LOGGER_TABLE_NAME`. En `config/`: `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_REDIRECT_URI`, `SCOUT_*`, `TYPESENSE_*`, `SMS_AUTH`.

## Pruebas

PHPUnit 11 (unos 31 archivos `*Test.php`). `Tests\TestCase` hace auto-seed y ofrece `createAdminUser()`.

```bash
php artisan test
php artisan test --filter test_create_user
```

`CLAUDE.md` dice que las pruebas corren contra Postgres (`fidudb_test` en `postgres-test`), pero el `phpunit.xml` actual declara `DB_CONNECTION=sqlite` y `:memory:`. Conviene verificar antes de confiar en cualquiera de los dos. CI: `azure-pipeline.yaml` (develop/qa, etapa `ValidateAndTest`) y `azure-pipeline-pr-review.yaml` (revisión de PR). Hay escaneo con Bearer (`bearer.yml`) y Sonar (`sonar-project.properties`).

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `develop` |
| Último commit | `4c2ad2f` (2026-08-26) Merged PR 216375: feat: update TreasuryPermissions to include additional actions for integration parameters => DEV |
| Remoto | `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_users` |

Los cambios recientes son mayormente catálogos de permisos (Tesorería, Impuestos).

## Notas

- Los commits deben seguir Conventional Commits; lo valida el hook `.githooks/commit-msg`.
- Los permisos son jerárquicos: `module-user-<feature>` habilita el prefijo, y `-list`, `-show`, `-edit`, `-create`, `-delete`, `-export` y `-action_*` habilitan cada acción.
- Este repo también siembra los permisos de los demás módulos del ERP (por ejemplo, `TreasuryPermissions` y `TaxPermissions`).
- Hay archivos sobrantes: `docker-dev/php-fpm/Dockerfile copy`.
