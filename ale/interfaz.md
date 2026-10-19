# Linux Quest (interfaz)

**Ruta local:** `/Users/johnhernandez/Documents/Ale/interfaz`

## Descripción / propósito

Juego educativo web, **"Linux Quest — Aprende la terminal jugando"**: una terminal Linux simulada en el navegador con misiones narrativas, objetivos verificables, pistas, teoría, experiencia (XP), rangos y logros.

## Stack

HTML5 + CSS + JavaScript vanilla (sin dependencias ni build). ~3.800 líneas en total.

## Estructura

```
interfaz/
├── index.html        # Layout: barra superior (XP, rango, logros, tema, reinicio), panel de misiones, terminal
├── css/style.css     # Estilos y tema claro/oscuro
└── js/
    ├── fs.js         # Sistema de archivos virtual en memoria
    ├── commands.js   # Implementación de comandos (defcmd)
    ├── levels.js     # Misiones: historia, objetivos, pistas, teoría
    └── game.js       # Motor: terminal, verificación de objetivos, progreso, rangos
```

## Funcionalidades

- **17 misiones**: Aterrizaje forzoso, Aprender a caminar, Lo que no se ve, Monta tu taller, Copiar/mover/renombrar, El comando sin vuelta atrás, Redirigir el caudal, Editor a bordo, Cazar la aguja, Lo que se perdió en el árbol, Medir antes de tocar, Fontanería: las tuberías, Quién puede hacer qué, El disco lleno, Procesos fuera de control, Examen final: turno de guardia, Modo libre.
- **Comandos simulados (~50):** `pwd cd ls tree cat less head tail touch mkdir rmdir cp mv rm ln echo grep find sort uniq wc file stat chmod chown du df free ps kill nano man help history which whoami id hostname uname date env export sudo clear exit cowsay neofetch`, con redirecciones y tuberías.
- Rangos: Novato → Aprendiz → Usuario → Operador → Administrador → Sysadmin → Pingüino mayor. Procesos simulados (para `ps`/`kill`).
- Progreso persistido en `localStorage` (clave `linuxquest.v1`).

## Datos e integraciones

Ninguna; todo corre en el navegador.

## Cómo ejecutarlo

Abrir `index.html` en el navegador, o servir la carpeta: `python3 -m http.server` dentro de `interfaz/`.

## Variables de entorno / pruebas

No aplica. Sin pruebas automatizadas (en `.claude/settings.local.json` hay rastros de scripts de prueba `test.js`/`ui.js` que ya no están en la carpeta).

## Estado git

No es un repositorio propio (queda dentro del repo git del home, sin rastrear).
