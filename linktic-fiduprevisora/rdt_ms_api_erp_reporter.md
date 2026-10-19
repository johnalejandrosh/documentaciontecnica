# rdt_ms_api_erp_reporter

> Ruta local: `/Users/johnhernandez/Documents/Linktic/FiduPrevisora/rdt_ms_api_erp_reporter`

## Descripción / propósito

**Microservicio de reportes** del ERP. Genera reportes de forma asíncrona (hoy solo en Excel `.xlsx`) consultando **directamente las bases de datos de otros microservicios**. Los enriquece con llamadas HTTP a otros módulos, guarda el archivo en S3 y avisa el cambio de estado por WebSocket.

Según `informe.md` (2026-08-26): hay **25 definiciones** registradas en `ReportTypeRegistry` (24 productivas y 1 plantilla `example` inactiva) que cubren 7 módulos origen:

| Módulo origen | Conexión | Reportes |
|---|---|---|
| business-trust | `micro_business_trust_pgsql` | 10 (pólizas, cesiones, negocios fiduciarios y su detalle, permisos, fideicomitentes/beneficiarios, garantías, proyecto inmobiliario, unidades vendidas, desistimientos) |
| accounts-payable | `micro_accounts_payable_pgsql` | 7 (estado de ORPA, instrucciones pagadas, autorizadores, pagos programados, solicitudes de pago, cierre, ORPA anuladas) |
| financial-obligations | `micro_financial_obligations_pgsql` | 2 |
| normative | `micro_normative_pgsql` | 2 |
| accountability | `micro_accountability_pgsql` | 1 |
| seizure | `micro_seizure_pgsql` | 1 |
| portal-payment | `micro_portal_payment_pgsql` | 1 |

Hay conexiones declaradas que todavía no tienen reportes: billing-collection, commission-settlement, fixed-assets, tax-service, treasuries y fics.

## Stack y versiones principales

| Componente | Versión |
|---|---|
| PHP | ^8.2 |
| Laravel | ^11.9 (lock v11.55.0) |
| fiduprevisora/core | dev-develop |
| Sanctum / Reverb | v4.3.2 / v1.10.2 |
| aws/aws-sdk-php + flysystem S3 | 3.387.2 / ^3.29 |
| keepsuit/laravel-opentelemetry | 2.2.3 |
| maatwebsite/excel, phpspreadsheet ^1.30.5 | sí |
| sqs-fifo-queue, predis | sí |
| Dev | PHPUnit 11, Larastan, Pint, Boost, Telescope, deptrac |

## Arquitectura y estructura de carpetas

```
app/
├── Reports/
│   ├── Contracts/        ReportDefinition (key/title/module/columns/rules/query/externalCalls/formats),
│   │                     BaseReportDefinition (table(), applyDateRange(), reglas de rango de fechas)
│   ├── Definitions/      una clase por reporte, agrupadas por módulo
│   ├── Formats/          ExcelWriter, ExcelStyler, FormatWriter, ReportFormat
│   ├── Support/          ExternalReportDataFetcher, ReportUserNameResolver, ThirdPartyDisplay, Catalogs
│   ├── ConnectionResolver.php   MicroserviceType → conexión Postgres
│   └── ReportTypeRegistry.php   mapa key → clase (fuente para el seeder)
├── Jobs/                 GenerateReportJob (3 intentos, backoff 30/120 s, timeout 300 s, ShouldBeUnique),
│                         SendExternalNotificationJob
├── Events/               ReportStatusUpdated (ShouldBroadcast)
├── ExternalModules/      Accounting, BusinessTrust, Fics, FinancialObligation, FixedAssets,
│                         InvestmentPortfolio, Notification, ThirdParty, Treasury, User
├── Console/Commands/     MakeReportCommand (make:report), PruneReportsCommand (reports:prune), ResetOpcache
└── Http, Models, Services/V1 (ReportService, S3Service), ...
resources/views/excel/    vistas Blade por reporte (Blade → HTML → PhpSpreadsheet → Xlsx)
openspec/                 schema `rdt-agent-driven`; changes: report-generation-standard y reporte-* (ORPA, pagos...)
docs/adr/                 plantilla de ADR
```

**Flujo**: `POST /reports` → `StoreReportRequest` → `ReportService::create()` (resuelve la definición, valida filtros, aborta si la consulta no trae datos, crea `Report`) → `GenerateReportJob` (consulta en streaming con `lazy()`, pool concurrente de `externalCalls`, escribe el Excel, lo sube a S3 en `reports/{module}/{key}_{fecha}.xlsx`, marca el reporte como completado o fallido y emite `ReportStatusUpdated`). La descarga entrega una URL prefirmada de S3 válida por 10 minutos.

## Módulos / endpoints principales

Prefijo `api/reporter` (`ExternalModule::_MODULES['reporter']`).

| Método | Ruta | Permiso |
|---|---|---|
| GET | `/example` | pública (bienvenida) |
| GET | `/select-tables` | sanctum (selectores `report_modules`, `report_types`, caché 300 s) |
| GET | `/reports` | `module-reporter-general-list` |
| POST | `/reports` | `module-reporter-general-action_generate` |
| GET | `/reports/{report}/download` | `module-reporter-general-export` |
| GET | `/business_trust_external_module` | sanctum (prueba de integración, heredada de la plantilla) |

Canal de broadcast: `reporter.report-status.user-{userId}`. La tarea `reports:prune` está comentada en `routes/console.php`.

## Dependencias e integraciones

- **PostgreSQL**: `micro_reporter_pgsql` (BD propia: reportes, tipos, módulos) más conexiones de solo lectura a las BD de business_trust, accounts_payable, financial_obligations, normative, accountability, seizure, portal_payment, billing_collection, commission_settlement, fixed_assets, tax_service, treasuries, fics y users.
- **AWS S3**: almacenamiento de los reportes y URLs prefirmadas.
- **Colas**: `GenerateReportJob`, con conexión configurable (`QUEUE_CONNECTION`, `DB_QUEUE_CONNECTION`). Drivers database, redis, sqs y sqs-fifo. `DISPATCH_SYNC_JOB` permite ejecutarlo de forma síncrona.
- **Reverb / broadcasting**: avisa al frontend el estado del reporte.
- **Microservicios HTTP**: terceros, usuarios, investment-portfolio (monedas), fixed-assets, financial-obligations, fics (oficinas), treasury, accounting, notificaciones.
- **OpenTelemetry**.

## Cómo ejecutarlo localmente

Desde el orquestador (perfil opcional `reporter`):

```bash
cd ../rdt_ms_api_erp_core && ./startup.v2.sh reporter
docker exec -it fidu-reporter php artisan migrate --seed
docker exec -it fidu-reporter php artisan make:report BUSINESS_TRUST TrustBalance
docker exec -it fidu-reporter php artisan reports:prune --days=90
```

El `README.md` describe además un modo independiente heredado de la plantilla "Proyecto BASE":

```bash
cp .env.example .env
docker compose up -d --build                     # nginx 84, postgres 5436
docker exec -it project_base_php bash
php artisan migrate --database=base_pgsql --seed
docker network connect rdt_ms_api_erp_core_app-network project_base_php
```

## Variables de entorno (solo nombres)

`APP_NAME`, `APP_ENV`, `APP_BUSINESS_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_TIMEZONE`, `APP_URL`, `API_ENVIRONMENT`, `BASE_ENDPOINT_API`, `API_EXTERNAL`, `APP_LOCALE`, `APP_FALLBACK_LOCALE`, `APP_FAKER_LOCALE`, `APP_MAINTENANCE_*`, `BCRYPT_ROUNDS`, `LOG_*`, `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`. Conexiones por microservicio, cada una con `_HOST`, `_PORT`, `_DATABASE`, `_USERNAME` y `_PASSWORD`: `DB_MICRO_REPORTER_PGSQL_*`, `DB_MICRO_BUSINESS_TRUST_PGSQL_*`, `DB_MICRO_BILLING_COLLECTION_PGSQL_*`, `DB_MICRO_COMMISSION_SETTLEMENT_PGSQL_*`, `DB_MICRO_SEIZURE_PGSQL_*`, `DB_MICRO_FIXED_ASSETS_PGSQL_*`, `DB_MICRO_FINANCIAL_OBLIGATIONS_PGSQL_*`, `DB_MICRO_TAX_SERVICE_PGSQL_*`, `DB_MICRO_ACCOUNTABILITY_SERVICE_PGSQL_*`, `DB_MICRO_ACCOUNTS_PAYABLE_PGSQL_*`, `DB_MICRO_NORMATIVE_PGSQL_*`, `DB_MICRO_PORTAL_PAYMENT_SERVICE_PGSQL_*`. Además: `LDAP_*`, `BROADCAST_CONNECTION`, `FILESYSTEM_DISK`, `QUEUE_CONNECTION`, `MEMCACHED_HOST`, `CACHE_STORE`, `CACHE_DRIVER`, `REDIS_*`, `CACHE_PREFIX`, `MAIL_*`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, `AWS_USE_PATH_STYLE_ENDPOINT`, `VITE_APP_NAME`, `PAGINATE`, `AUTH_GUARD`, `AZURE_TOKEN`, `REVERB_APP_ID`, `REVERB_APP_KEY`, `REVERB_APP_SECRET`, `REVERB_HOST`, `REVERB_PORT`, `REVERB_SCHEME`, `REPORTS_RETENTION_DAYS`, `SERVICES_INTERNAL_TOKEN`, `DISPATCH_SYNC_JOB`, `DB_QUEUE_CONNECTION`.

## Pruebas

PHPUnit 11 (unos 54 archivos): 25 pruebas feature de reporte, `ReportApiTest` y 19 pruebas unitarias de definiciones. La BD de pruebas es SQLite en archivo (`database/testing.sqlite`). CI (`azure-pipeline.yaml`, develop/qa): `ValidateAndTest`. Escaneo con Bearer.

## Estado git

| Campo | Valor |
|---|---|
| Rama actual | `chore/report-library` (no es `develop`) |
| Último commit | `4c27ab8` (2026-08-25) Merged PR 215803: fix: format generated_at without seconds (Y-m-d H:i) |
| Remoto | `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_reporter` |
| Cambios locales | `informe.md` sin versionar |

## Notas

- `ARCHITECTURE.md` y `README.md` están **desactualizados**: describen la plantilla base (ExampleController, S3). La descripción real del módulo de reportes está en `informe.md`.
- Las definiciones usan literales con el FQCN del modelo origen (por ejemplo `attachable_type`). Si se renombra la clase en el micro origen, la columna queda vacía sin que falle nada.
- Si falla una sola llamada externa, el reporte completo queda como fallido.
- En el orquestador está registrado como `reporter:reporter:rdt-reporter:reporter:optional` en `services.conf`. En `docker-compose.v2.yml` aparece como servicio `rdt-reporter` (contenedor `fidu-reporter`), con su propia BD `postgres-reporter` (`reporter_db`).
