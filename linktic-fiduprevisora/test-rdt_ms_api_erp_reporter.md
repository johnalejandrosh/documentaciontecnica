# Test/rdt_ms_api_erp_reporter (copia)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/Test/rdt_ms_api_erp_reporter`
> Versión principal: [rdt_ms_api_erp_reporter.md](rdt_ms_api_erp_reporter.md)

## Descripción / propósito

Es una **copia del microservicio de reportes `rdt_ms_api_erp_reporter`**, en la misma rama y el mismo commit que la versión de `FiduPrevisora/`. El propósito, el stack (Laravel 11, Reverb, S3, colas), la arquitectura (`app/Reports`), los endpoints (`/reports`, `/select-tables`), las variables de entorno y las pruebas son **idénticos** a los de la versión principal.

## Diferencias con la versión de FiduPrevisora

| Aspecto | FiduPrevisora/rdt_ms_api_erp_reporter | Test/rdt_ms_api_erp_reporter |
|---|---|---|
| Rama | `chore/report-library` | `chore/report-library` |
| Último commit | `4c27ab8` (2026-08-25) | `4c27ab8` (2026-08-25), **el mismo** |
| Remoto | SSH `git@ssh.dev.azure.com:v3/Linktic/384%20-%20CORE%20FIDUPREVISORA/rdt_ms_api_erp_reporter` | **HTTPS** `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_reporter` |
| `vendor/`, `core-package/` | presentes | **ausentes** |
| `informe.md` (inventario de reportes, sin versionar) | presente | ausente |
| Artefactos locales (`.phpunit.result.cache`, `bootstrap/cache/*.php`, `database/testing.sqlite`, `docker/postgres/`) | presentes | ausentes |
| `.env` | local | distinto |
| Working tree | `informe.md` sin versionar | limpio |

## Cómo ejecutarlo localmente

Igual que la versión principal, desde el orquestador. Hay que tener en cuenta que el orquestador hermano en `Test/rdt_ms_api_erp_core` no tiene el resto de los servicios obligatorios clonados.

## Estado git

- Rama: `chore/report-library`
- Último commit: `4c27ab8` Merged PR 215803: fix: format generated_at without seconds (Y-m-d H:i)
- Remoto: `https://Linktic@dev.azure.com/Linktic/384%20-%20CORE%20FIDUPREVISORA/_git/rdt_ms_api_erp_reporter`

## Notas

Es un duplicado sin dependencias instaladas. La versión de trabajo activa es la de `FiduPrevisora/`.
