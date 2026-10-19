# my-app

> Ruta local: `/Users/johnhernandez/Documents/Linktic/projectos_alter/my-app`

## Descripción / propósito

Proyecto **Next.js recién generado con `create-next-app`**, sin código propio: solo la página de bienvenida por defecto. Por fechas (5 de octubre de 2026, el mismo día en que arrancó `salud-apps`) y por compartir versiones exactas (Next 16.3.8, React 19.2.8, pnpm 11.20.0) parece ser el **scaffold de prueba o punto de partida de [salud-apps](salud-apps.md)**.

## Stack y versiones

- Next.js `16.3.8` (App Router), React / React DOM `19.2.8`
- TypeScript `^5`, Tailwind CSS `^4` (`@tailwindcss/postcss`), ESLint `^9` + `eslint-config-next 16.3.8`
- pnpm `11.20.0`

## Arquitectura y estructura de carpetas

```
app/       layout.tsx, page.tsx, globals.css, favicon.ico
public/    SVGs por defecto (next, vercel, file, globe, window)
AGENTS.md  bloque de reglas de Next.js para agentes ("This is NOT the Next.js you know")
CLAUDE.md  apunta a AGENTS.md
```

## Funcionalidades / pantallas

Solo `/` con la página de ejemplo de Next.js.

## Datos e integraciones

Ninguna.

## Cómo ejecutarlo localmente

```bash
pnpm install
pnpm dev     # http://localhost:3000
```

## Variables de entorno

Ninguna.

## Pruebas

No hay pruebas.

## Estado git

No es un repositorio propio (solo queda bajo el repositorio del home `/Users/johnhernandez`, sin commits). Sin remoto.

## Notas

Candidato a eliminar si ya no se usa; el trabajo real está en `salud-apps`.
