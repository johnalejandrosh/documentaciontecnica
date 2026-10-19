# ETL Data Core (02_etl_data_core)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/proyctos_externos/proyectos_externos/02_etl_data_core`

## Descripción / propósito

Script ETL en Python que trae los **casos de mesa de ayuda de la API Data Core** (marcas `balu`, `docum`, `fiscalia` y `rama`) a la base del [Tablero Proyectos Externos](tablero-proyectos-externos.md) y refresca el rollup unificado que lee el tablero. Sin esta ETL, los tableros de BALÚ, SGDEA (Docum) y Fiscalía se congelan desde el 1-ago-2026. Forma parte del paquete de entrega del 13-oct-2026, adaptado para correr en cualquier servidor con cron.

## Stack y versiones

- Python >= 3.10 (el `venv/` local usa Python 3.14)
- `psycopg2-binary >=2.9,<3`, `requests >=2.31,<3`
- Un solo archivo: `etl_data_core.py` (~37 KB)
- Cron del servidor para la programación

## Arquitectura y estructura de carpetas

```
etl_data_core.py          ETL completa (modos, candado, bitácora, refresco de rollup)
sql/01_tablas_ingesta.sql crea data_core_cases y data_core_ingesta_log (idempotente)
docs/analisis.md          análisis de la base de destino (19-oct-2026)
README.md                 instalación, cron, decisiones, trampas, monitoreo
.env.example, requirements.txt, venv/ (local)
```

Flujo: `API Data Core → data_core_cases (UPSERT por (modulo, external_case_id), nunca borra) → vista data_unificado → data_rollup_unificado* → tablero`. Cada corrida escribe una fila por marca en `data_core_ingesta_log`.

## Funcionalidades (modos)

| Modo | Cuándo | Qué hace |
|---|---|---|
| `sync` (defecto) | Cada hora | Trae lo cambiado desde la última corrida exitosa (con solape de 6 h) y refresca mes actual y anterior |
| `rebarrer` | Diario | Vuelve a pedir todo desde el corte; recupera cambios publicados con retraso |
| `conciliar` | Diario | Compara conteos API vs tabla por marca y mes; escribe faltantes, avisa sobrantes |
| `apertura` | Manual | Recarga meses completos por fecha de apertura (`--desde/--hasta YYYY-MM`) |
| `refrescar` | Manual | Solo refresca el rollup, sin llamar a la API |
| `verificar` | Instalación | Revisa base, tablas, funciones y token; no escribe |

Opciones: `--marca <marca>`, `--sin-refresco`, `--desde`, `--hasta`.

## Datos e integraciones

- **API Data Core** (OAuth client credentials). Las credenciales no vienen en el paquete; llegan por correo oficial.
- **PostgreSQL / Supabase**, esquema `tablero_proyectos_externos`, con el usuario dueño de las tablas (RLS activo sin políticas).
- Webhook opcional de alertas (Google Chat o Slack).

## Cómo ejecutarlo localmente

```bash
python3 -m venv venv && ./venv/bin/pip install -r requirements.txt
psql "$DATABASE_URL" -f sql/01_tablas_ingesta.sql
cp .env.example .env && chmod 600 .env
./venv/bin/python etl_data_core.py --modo verificar       # debe terminar en "Resultado: todo OK"
./venv/bin/python etl_data_core.py --modo sync --desde 2026-08-01   # carga inicial (~150.000 casos, 2-3 min)
```

Cron sugerido (hora Colombia): `sync` cada hora (minuto 47), `rebarrer` 04:53, `conciliar` 05:23.

## Variables de entorno (solo nombres)

Obligatorias: `DATABASE_URL`, `DATA_CORE_CLIENT_ID`, `DATA_CORE_CLIENT_SECRET`.
Opcionales: `DATA_CORE_API_BASE`, `DB_SCHEMA`, `DATA_CORE_MARCAS`, `ALERTA_WEBHOOK_URL`, `LOG_FILE`, `LOCK_FILE`.

Hay `.env` y `.env.local-db` locales con valores (permisos 600): no compartir.

## Pruebas

Sin pruebas automatizadas; la verificación es `--modo verificar` y la bitácora `data_core_ingesta_log` (ver sección 7 del README).

## Estado git

No es un repositorio propio (queda bajo el repositorio del home `/Users/johnhernandez`, sin commits). `.gitignore` local excluye los `.env`.

## Notas

- Decisiones clave: corte 2026-08-01 en hora Bogotá; actualización por `source_updated_at`; refresco de rollup un mes por llamada (por `statement_timeout`, error 57014); RAMA sigue por Excel; lo variable por marca vive en `modulo_fuente_api`.
- `docs/analisis.md` (19-oct-2026): la API **dejó de traer datos el 2026-10-16** (error DNS: el API Gateway anterior ya no existe); 13 tablas, 1 vista, 12 funciones; problemas de calidad (mayúsculas de nivel distintas entre fuentes, `departamento` ausente en la API, casos duplicados en `data`).
- Existe un `.etl_data_core.lock` (vacío) de la última corrida local.
- La credencial actual tiene permiso `admin:sync:write`; no usar `/sync/reopen` ni `/sync/complete`. Se recomienda pedir una credencial propia.
