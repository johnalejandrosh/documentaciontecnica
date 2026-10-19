# Asistencia a talleres

**Ruta local:** `/Users/johnhernandez/Documents/Ale/asistencia`

## Descripción / propósito

Aplicación web para registrar la asistencia a talleres mediante código QR. La persona asistente escanea el QR del taller, deja sus datos (documento, contacto, autorizaciones de contacto), acepta la política de tratamiento de datos y califica la sesión (1 a 5 estrellas + comentarios de mejora). Un panel de administración (`/admin`) gestiona talleres, registros de asistencia, personas, usuarios, roles y permisos. Pertenece al grupo GitLab `astrociencia-y-tecnologia`.

## Stack y versiones

| Componente | Versión |
| --- | --- |
| Next.js (App Router, `proxy.ts` en vez de middleware) | 16.3.0 |
| React / React DOM | 19.2.8 |
| Prisma (ORM + `@prisma/adapter-pg`) | ^7.9.1 |
| PostgreSQL | 17 (imagen `postgres:17-alpine`) |
| Tailwind CSS | 4 |
| TypeScript | 5 |
| `jose` (JWT) | ^6.2.8 |
| `qrcode` | ^1.5.4 |
| Node (Docker) | 22-alpine |
| Gestor de paquetes | pnpm 11.20.0 |

## Arquitectura y estructura de carpetas

```
asistencia/
├── app/                 # Rutas (App Router)
│   ├── page.tsx         # Portada pública: listado de talleres
│   ├── talleres/[slug]/ # Página pública del taller + formulario de asistencia
│   ├── ingresar/        # Login del panel
│   ├── politicas/       # Política de tratamiento de datos
│   ├── admin/           # Panel (talleres, asistencias, asistentes, usuarios, roles, perfil)
│   └── api/             # salud, sesion/refrescar
├── components/          # UI pública + components/admin (formularios, tablas, QR)
├── lib/                 # Server Actions (acciones-*.ts), datos-*.ts, validación, permisos
│   ├── auth/            # tokens JWT, sesiones, contraseñas (scrypt)
│   └── generated/prisma # Cliente Prisma generado
├── prisma/              # schema.prisma, migraciones, seeds (demo, auth), purga de sesiones
├── proxy.ts             # Puerta del panel (verifica JWT en Edge)
├── Dockerfile, docker-compose*.yml, .gitlab-ci.yml
└── public/subidas/      # Portadas de talleres subidas desde el panel
```

- **Autenticación propia:** access token JWT (firmado con `AUTH_SECRET`) + refresh token rotativo guardado como hash SHA-256 en la tabla `sesiones`, con detección de reuso por "familia" de tokens. Contraseñas con `scrypt`. Cambio de contraseña forzado al primer ingreso.
- **Autorización:** RBAC. El catálogo de permisos vive en código (`lib/permisos.ts`, grupos: talleres, asistencias, asistentes, usuarios, roles) y se siembra en BD; los roles se crean desde el panel. Roles iniciales: Administrador, Coordinación, Consulta.
- **Almacenamiento de archivos:** disco local (`public/subidas/`), pensado para servidor propio (volumen Docker `subidas`).

## Funcionalidades / pantallas / endpoints principales

**Público**
- `/` — listado de talleres activos.
- `/talleres/[slug]` — detalle del taller y formulario de registro de asistencia + calificación.
- `/politicas` — política de datos (se guarda la versión aceptada en cada registro).
- `/ingresar` — inicio de sesión.

**Panel (`/admin`)**
- Talleres: listar, crear, editar, activar/ocultar, eliminar, portada, ver y descargar QR (`/admin/talleres/[id]/qr/descargar`).
- Asistencias: listado con filtro por taller, edición de calificación/comentarios.
- Asistentes (personas): listado y edición.
- Usuarios: CRUD, activar/desactivar, restablecer contraseña (credenciales temporales), sesiones.
- Roles: CRUD con matriz de permisos.
- Perfil y cambio de contraseña.

**API**
- `GET /api/salud` — healthcheck del contenedor (no toca la BD).
- `/api/sesion/refrescar` — rotación del refresh token.

## Datos e integraciones

PostgreSQL vía Prisma. Modelos: `Usuario`, `Rol`, `Permiso`, `RolPermiso`, `Sesion`, `Taller`, `Asistente`, `Asistencia`. Enums: `TipoDocumento` (CC, TI, CE, PA, PEP, NIT, RC), `Modalidad` (PRESENCIAL, VIRTUAL, HIBRIDO). Restricciones clave: asistente único por (tipo, número de documento); una asistencia por persona y taller. Migraciones en `prisma/migrations/` (init, contacto y consentimiento, autenticación, portada y enlace). Sin integraciones externas.

## Cómo ejecutarlo localmente

Con Docker (recomendado, según README):

```bash
cp .env.example .env              # completar variables
docker compose up -d --build      # app en http://localhost:3000
docker compose run --rm herramientas pnpm db:seed:auth   # primer admin (imprime contraseña temporal)
docker compose run --rm herramientas pnpm db:seed        # datos demo (opcional)
```

Sin Docker (desarrollo): `pnpm install` (corre `prisma generate`), `pnpm db:migrate`, `pnpm db:seed:auth`, `pnpm dev`. Otros scripts: `db:studio`, `db:purgar-sesiones`, `db:seed:limpiar`, `lint`, `build`.

Servicios de `docker-compose.yml`: `base` (Postgres), `migraciones`, `app`, `herramientas`; volúmenes `datos-postgres` y `subidas`. Existe además `docker-compose.produccion.yml`.

## Variables de entorno (solo nombres)

`DATABASE_URL`, `AUTH_SECRET`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `ADMIN_CORREO`, `ADMIN_CONTRASENA` (opcional), `PUERTO` (opcional), `NEXT_PUBLIC_APP_URL` (opcional), `REGISTRO_BASE` (opcional).

Variables de CI/CD en GitLab: `SSH_LLAVE_PRIVADA`, `SSH_HOSTS_CONOCIDOS`, `SSH_USUARIO`, `SSH_HOST`, `RUTA_DESPLIEGUE`.

## Pruebas

No hay pruebas automatizadas (ni unitarias ni e2e). La verificación se hace en el pipeline de GitLab (`.gitlab-ci.yml`), etapa `verificar`: lint, tipos, build y aplicación de migraciones contra una base limpia. Etapas siguientes: `construir` (imágenes app y herramientas al Container Registry) y `desplegar` (por SSH: pull, migraciones y arranque).

## Estado git

- Repositorio propio. Rama: `main`.
- Remoto: `http://gitlab.johnr.top/astrociencia-y-tecnologia/asistencia.git`
- Últimos commits:
  - `bcc163a` adjust
  - `72d66d4` adjust docker
  - `bf062e8` update readme
  - `17565d1` implementacion de modulo usuarios, roles, CI/CD
  - `ad46d0a` config adjust
  - `8f3eff7` Initial commit
  - `f7e4bce` Initial commit from Create Next App
- Cambios sin commit: `README.md` modificado.

## Notas

- `AGENTS.md`/`CLAUDE.md` advierten que Next 16 tiene cambios de API: consultar `node_modules/next/dist/docs/` antes de modificar código.
- Existe un `.env` local (ignorado por git) además de `.env.example`.
- Código y nombres de dominio en español (acciones, datos, validación).
- Hay un script suelto `.probar-portada.ts` en la raíz.
