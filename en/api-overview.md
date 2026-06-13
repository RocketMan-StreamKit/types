# API overview

All APIs are **global** in the integration worker VM. Source of truth for typings: `data/addon.d.ts` (generated from sandbox JSDoc).

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
| `notify` | Title-bar notification center |
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
- [Utilities](./api-utilities.md)
