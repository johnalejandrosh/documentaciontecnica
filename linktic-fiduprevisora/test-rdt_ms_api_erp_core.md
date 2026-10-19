# Test/rdt_ms_api_erp_core (copia)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Test/rdt_ms_api_erp_core`
> Versión principal: [rdt_ms_api_erp_core.md](rdt_ms_api_erp_core.md)

## Descripción / propósito

Es una **copia de trabajo del orquestador `rdt_ms_api_erp_core`**, al mismo commit que la versión de `FiduPrevisora/`. Parece un clon de prueba (creado el 2026-08-26), quizá para probar el clonado por HTTPS o un arranque limpio sin dependencias instaladas. El propósito, el stack, la arquitectura, los endpoints, las variables de entorno y las pruebas son **idénticos** a los de la versión principal.

## Diferencias con la versión de FiduPrevisora

| Aspecto | FiduPrevisora/rdt_ms_api_erp_core | Test/rdt_ms_api_erp_core |
|---|---|---|
| Rama | `develop` | `develop` |
| Último commit | `4a5cf3da` (2026-08-26) | `4a5cf3da` (2026-08-26), **el mismo** |
| Remoto | SSH `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_core` | **HTTPS** `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_core` |
| `vendor/`, `core-package/`, `logs/` | presentes | **ausentes** (no se ha instalado ni levantado) |
| `nginx/sites/default.conf`, `nginx/ssl/` | presentes (generados) | **ausentes** (no se ha corrido `generate_nginx_config.v2.sh`) |
| `.env` | local | distinto (configuración local propia) |
| Árbol de código versionado | igual | igual (`git status` limpio) |

## Cómo ejecutarlo localmente

Igual que la versión principal (`./generate_nginx_config.v2.sh` y luego `./startup.v2.sh`). Hay que tener en cuenta que los scripts buscan repos hermanos en el directorio padre (`../rdt_ms_api_erp_*`). En `Test/` solo existe además `rdt_ms_api_erp_reporter`, así que faltarían los servicios obligatorios (users, security, thirdparties, broadcasting) y `startup.v2.sh` fallaría.

## Variables de entorno

Las mismas que la versión principal (ver [rdt_ms_api_erp_core.md](rdt_ms_api_erp_core.md)).

## Estado git

- Rama: `develop`
- Último commit: `4a5cf3da` Merged PR 216620: feat: make has_cash_flow nullable in business_trust_treasuries table - HU0010 HU0057
- Remoto: `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_core`
- Working tree limpio.

## Notas

Es un duplicado; si no se está usando para algo puntual, se puede eliminar.
