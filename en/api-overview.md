# API overview

All APIs are **global** in the integration worker VM. Typings ship in the npm package `@rocketman-streamkit/types` (`addon.d.ts`).

## Globals

| Global | Purpose |
| --- | --- |
| `isDeveloperMode` | `true` in dev builds or when user enables Developer mode |
| `LANG` | `current` (`en` / `ru` / `uk`), `onChangeLanguage(cb)` — app UI locale bridge |
| `permissions` | `list`, `has(permission)` |
| `data` | Instance metadata: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — bind HTTP/Socket.IO callbacks |
| `network` | HTTP client, inbound routes, WebSocket, Socket.IO |
| `api` | `openUrl`, `restart`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest` — addon-to-addon RPC |
| `dashboard` | Events widget, chat, platforms, overlay triggers |
| `status` | Main window status bar |
| `viewers` | Live viewer count in the main window status bar (no permission) |
| `notify` | Title-bar notification center |
| `tts` | Text-to-speech playback (`TTS` permission) |
| `storage` | Private JSON file in install folder |
| `ui.auth` | OAuth success/fail redirect URLs |
| `crypto` | `createPkce`, `verifyRsaSha256` |
| `console` | Prefixed logging |
| `setTimeout` / `setInterval` / `clearTimeout` / `clearInterval` | Clamped timers with error isolation |
| `sleep(ms)` | Promise delay (max 60 s) |
| `random` | `number(min, max)`, `id()` |
| `URL`, `URLSearchParams` | Sandbox-safe URL helpers |
| `GenerateConfig(schema)` | Register settings schema |
| `require(name)` | Node require — **ROOT only** |

## Documentation map

- [events](./api-events.md)
- [network](./api-network.md)
- [api.config & storage](./api-config-storage.md)
- [addons RPC](./api-addons-rpc.md)
- [dashboard](./api-dashboard.md)
- [status, notify, ui](./api-status-notify-ui.md)
- [Text-to-speech (`tts`)](./api-tts.md)
- [Utilities](./api-utilities.md)
