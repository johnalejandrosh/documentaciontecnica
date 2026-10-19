# testfidu/rdt_ms_api_erp_core (clon antiguo en `master`)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/testfidu/rdt_ms_api_erp_core`
> Versión principal: [rdt_ms_api_erp_core.md](rdt_ms_api_erp_core.md)

## Descripción / propósito

Es un clon por **SSH** del repo `rdt_ms_api_erp_core` que se hizo el 2026-08-26 para verificar el procedimiento de la guía `testfidu/GUIA-CLONE-SSH-AZURE.md`: llave SSH dedicada `~/.ssh/id_rsa_azure`, alta en Azure DevOps y `~/.ssh/config` con `IdentitiesOnly yes`. Tiene checkout de la rama **`master`**, que en el remoto quedó congelada en febrero de 2025. Por eso muestra una **versión muy temprana del core**: el "core base" con las primeras migraciones y los modelos de agenda y notificaciones. No es la versión de trabajo actual (`develop`).

## Diferencias con la versión de FiduPrevisora

| Aspecto | FiduPrevisora/rdt_ms_api_erp_core | testfidu/rdt_ms_api_erp_core |
|---|---|---|
| Rama | `develop` | **`master`** |
| Último commit | `4a5cf3da` (2026-08-26) | **`b7bc9dc5` (2025-02-20)** "Modelos y database del proyecto agenda y notificaciones" |
| Commits en la rama | ≈3.716 | 11 |
| Remoto | SSH (`git@ssh.dev.azure.com:v3/Linktic/...`) | el mismo remoto SSH (`origin/develop` está disponible, pero sin checkout) |
| Migraciones | ≈1.246 | 233 |
| `fiduprevisora/core` | `dev-develop`, desde el repo de Azure | `dev-main` |
| `phpspreadsheet` | ^1.30.5, más `symfony/http-client` | ^1.29 |
| `app/` | 16 carpetas (Services, Traits, Exports, etc.) | solo `Http`, `Imports`, `Models` (Asset, Main, Notifications, Traits, User) |
| Orquestación | `docker-compose.v2.yml`, `services.conf`, `startup.v2.sh`, unos 33 servicios | `docker-compose.yml` y `docker-compose.prod.yml` mínimos: load balancer nginx, `rdt-users`, `rdt-core` / `rdt-migrations`, `postgres` |
| Ausentes | — | `AGENTS.md`, `docs/`, `infra/`, `scripts/`, `tests/`, `services.conf`, `docker-dev/`, pipelines |
| Variables de entorno | incluyen `DB_DATABASE`, `DB_MICRO_*`, `AZURE_TOKEN` | usa `DB_SERVICE_NAME` en lugar de `DB_DATABASE`; no tiene `DB_MICRO_*` ni `AZURE_TOKEN` |

## Stack

PHP ^8.2 y Laravel ^11.9, con Sanctum, spatie (permission, activitylog, query-builder, pdf), excel, dompdf, google2fa y `fiduprevisora/core` dev-main.

## Endpoints

Ninguno: `routes/app*.php` vacíos y `web.php` con la vista `welcome`. En `console.php` están las mismas cuatro tareas programadas de usuarios y roles que en `develop`.

## Cómo ejecutarlo localmente

No se recomienda: es un estado histórico. Para usar la versión vigente:

```bash
git -C /Users/johnhernandez/Documents/Linktic/testfidu/rdt_ms_api_erp_core checkout develop
```

Si no, conviene usar directamente `FiduPrevisora/rdt_ms_api_erp_core`.

## Estado git

- Rama: `master`
- Último commit: `b7bc9dc5` (2025-02-20) Modelos y database del proyecto agenda y notificaciones (Oscar Julian Rojas)
- Remoto: `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_core`
- Working tree limpio.

## Notas

- Su valor principal es la guía `GUIA-CLONE-SSH-AZURE.md` que está al lado, que explica cómo clonar repos de Azure DevOps (organización Linktic) por SSH.
- La rama por defecto del remoto (`master`) está desactualizada. Al clonar repos del proyecto hay que hacer checkout de `develop`.
