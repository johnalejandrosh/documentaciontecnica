# Mercadito (list-shopping) — app móvil

**Ruta local:** `/Users/johnhernandez/Documents/Ale/projects/list-shopping`

## Descripción / propósito

App móvil de **lista de mercado inteligente y offline-first** para el mercado colombiano, con nombre comercial **Mercadito**. Cubre el ciclo de compra completo: armar varias listas, estimar el costo, marcar productos en el almacén con un toque, agregar imprevistos, registrar precio real por item, cerrar la compra (estimado vs. real) y consultar el historial. Incluye catálogo de productos con autocompletado e historial de precios con gráfica, y (en desarrollo) un **asistente IA por chat y voz** que modifica la lista.

## Stack y versiones

| Componente | Versión |
| --- | --- |
| Expo SDK | ~57.0.25 |
| React Native | 0.86.3 |
| React | 19.2.3 |
| Expo Router (typed routes, React Compiler activado) | ~57.0.23 |
| expo-sqlite | ~57.0.3 |
| expo-audio (dictado por micrófono) | ~57.0.5 |
| react-native-reanimated / gesture-handler | 4.5.1 / ~2.32.0 |
| react-native-reorderable-list | ^0.18.1 |
| openai (SDK) | ^7.25.0 |
| TypeScript | ~6.0.3 |
| Gestor de paquetes | npm |

Identificadores: `com.mercadito.app` (iOS/Android), scheme `mercadito`, owner EAS `johnalejandrosh`.

## Arquitectura y estructura de carpetas

Arquitectura en capas sin backend: UI (Expo Router, tabs + stack) → repositories (SQL directo) → SQLite local.

```
list-shopping/
├── src/app/              # Rutas (cada archivo es una pantalla)
│   ├── (tabs)/           # home, index (Listas), history, settings
│   ├── list/[id].tsx     # Detalle de una lista
│   ├── list-chat.tsx     # Chat con el agente IA de la lista
│   ├── add-item.tsx, product-detail.tsx, products.tsx,
│   ├── purchase-detail.tsx, onboarding.tsx
├── src/components/       # UI (home/, chat/, item-row, price-chart, toast, ...)
├── src/db/database.ts    # Conexión y migraciones (PRAGMA user_version)
├── src/repositories/     # item, list, product, purchase, section, settings, stats
├── src/services/         # openai-service.ts, list-agent.ts
├── src/config/env.ts     # Lectura de la API key
├── assets/branding/      # Brief de diseño + copia del prototipo de Claude Design
└── app.json, eas.json
```

## Funcionalidades / pantallas principales

- **Tabs:** Home (dashboard analítico con estadísticas y gráfica de gasto), Listas, Historial, Configuración.
- **Listas múltiples** activas con progreso; detalle de lista con secciones, reordenamiento y swipe.
- **Items:** cantidad, unidad (unidad, libra, kilo, litro, arroba, paquete), precio estimado y real; creación múltiple.
- **Modo compra:** marcado con un toque, imprevistos, cierre de compra con snapshot en `purchases`/`purchase_items`.
- **Historial:** compras pasadas, detalle, clonar lista completa o solo pendientes.
- **Catálogo de productos** ("Mis productos") con autocompletado e historial de precios (`PriceChart`).
- **Onboarding** interactivo y splash animado; tema claro/oscuro; soporte web (`*.web.tsx`).
- **Asistente IA** (`list-chat`): agente con *tool calling* (`gpt-4o-mini`) que puede crear/editar/eliminar items, crear/renombrar secciones y mover items; dictado por voz transcrito con `whisper-1`.

## Datos e integraciones

- **SQLite local** (`expo-sqlite`): tablas `shopping_lists`, `products`, `sections`, `items`, `product_price_history`, `purchases`, `purchase_items`, `app_settings`. Migraciones incrementales con `ALTER TABLE` controladas por `PRAGMA user_version`.
- **OpenAI API** (chat completions con tools + `/v1/audio/transcriptions`), llamada directamente desde el cliente.
- **EAS** para builds (perfiles `development`, `preview` → APK, `production`).

## Cómo ejecutarlo localmente

```bash
npm install
cp .env.example .env     # definir la API key de OpenAI
npx expo start           # Expo Go / emulador / web
npx expo lint            # lint
npx tsc --noEmit         # typecheck
```

Al usar módulos nativos (audio, sqlite) puede requerirse un *development build*: `npx expo run:ios|android` o `eas build --profile development`.

## Variables de entorno (solo nombres)

`EXPO_PUBLIC_IA_APIKEY`

## Pruebas

No hay pruebas automatizadas. Validación con `expo lint` y `tsc --noEmit` (según `AGENTS.md`).

## Estado git

- Repositorio propio, **sin remoto configurado**.
- Rama actual: `feature/WI-016-024-multilista-productos-onboarding` (existe también `main`).
- Últimos commits:
  - `61bd116` Add multi-item creation and simplify item selection flow
  - `445709f` Fix tab icons, list layout, and conditional bottom bar
  - `836c585` Initial commit: Mercadito shopping list app
- Muchos cambios sin commit (~38 entradas): pantallas nuevas (home, settings, list/[id], list-chat, onboarding, products, product-detail), repositorios y servicios IA sin rastrear; `checkout.tsx` eliminado.

## Notas

- **Riesgo de seguridad:** la API key de OpenAI se expone en el bundle del cliente por usar prefijo `EXPO_PUBLIC_`. Para producción conviene un proxy/backend. El `.env` está ignorado por git.
- La documentación de producto, roadmap y work items (WI-001 a WI-036) está en `projects/knowledge` (ver `knowledge-mercadito.md`). WI-033 a WI-036 (OpenAI, secciones, dashboard, chat IA) están en *draft* pero ya tienen código.
- La knowledge base describe la app como "100% offline / sin APIs externas"; la integración con OpenAI rompe parcialmente esa premisa.
- `README.md` es el genérico de `create-expo-app`.
