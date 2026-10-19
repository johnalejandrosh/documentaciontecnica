# dev-bi-back-docum-v1 — Back de Docum BI (repositorio vacío)

- **Ruta local:** `/Users/johnhernandez/Documents/Linktic/Analitica/docum/dev-bi-back-docum-v1`
- **Tipo:** Back (placeholder)
- **Cliente / producto:** Positiva Compañía de Seguros · SGDEA "Docum"

## Descripción / propósito

Repositorio reservado para el backend de Docum BI. A la fecha **no contiene código**: solo el `README.md` plantilla que genera Azure DevOps ("Introduction / Getting Started / Build and Test / Contribute" con TODOs).

## Stack y versiones

No definido (sin `package.json` ni otros manifiestos).

## Arquitectura y estructura de carpetas

```
README.md   plantilla de Azure DevOps
```

## Funcionalidades / endpoints

Ninguno todavía.

## Fuentes de datos e integraciones

Ninguna implementada. Hoy el acceso a datos de Docum (BigQuery `oro_tableros`) lo hacen las API routes de `dev-bi-front-docum-v1`.

## Cómo ejecutarlo localmente

No aplica.

## Variables de entorno

Ninguna.

## Pruebas

Ninguna.

## Estado git

- **Remoto:** Azure DevOps `Linktic/BI Analytics/_git/dev-bi-back-docum-v1`
- **Rama:** `main`
- **Commits:** 1 — `dd0f5a5` Added README.md (Brayan Suarez Salazar, 2026-10-19)

## Notas

- Par natural de [`dev-bi-front-docum-v1`](./dev-bi-front-docum-v1.md). Si en el futuro se saca la lógica de datos del Next.js (provider BigQuery, motor de agregación, caché), este sería el destino.
