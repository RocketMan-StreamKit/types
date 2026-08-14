# API overview

All APIs are **global** in the integration worker VM. Typings ship in the npm package `@rocketman-streamkit/types` (`addon.d.ts`).

## Globals

| Global | Purpose |
| --- | --- |
| `isDeveloperMode` | `true` in dev builds or when user enables Developer mode |
| `ADDON_TMP_DIR` | Absolute per-addon scratch dir under OS temp (`…/StreamKitPlusAddons/{id}`); no consent for paths inside |
| `LANG` | `current` (`en` / `ru` / `uk`), `onChangeLanguage(cb)` — app UI locale bridge |
| `permissions` | `list`, `has(permission)` |
| `data` | Instance metadata: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — bind HTTP/Socket.IO callbacks |
| `network` | HTTP client, inbound routes, WebSocket, SSE, SignalR, Socket.IO |
| `api` | `openUrl`, `restart`, `getProcessStats`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest`, `emit`, `subscribe`, `getInfo` — addon-to-addon RPC, events, and install metadata |
| `dashboard` | Events widget, chat, platforms, overlay triggers |
| `status` | Main window status bar |
| `viewers` | Live viewer count in the main window status bar (no permission) |
| `alerts` | Cross-addon alert display state: `started` / `ended` / `isActive` / `onChange` (no permission) |
| `currency` | User's primary currency and amount conversion (no permission) |
| `language` | Detect text language via fastText lid.176 (~176 ISO codes; no permission) |
| `license` | License status and MD5 device key fingerprint (no permission) |
| `notify` | Title-bar notification center |
| `tts` | Text-to-speech playback and voice info (`TTS` permission) |
| `llm` | LLM Access chat completions (`LLM` permission) |
| `ytdlp` | Media downloads via bundled yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
| `storage` | Private JSON file in install folder |
| `ui.auth` | OAuth success/fail redirect URLs |
| `crypto` | `createPkce`, `verifyRsaSha256` |
| `console` | Prefixed logging |
| `setTimeout` / `setInterval` / `clearTimeout` / `clearInterval` | Timers with error isolation |
| `sleep(ms)` | Promise delay |
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
- [Alerts (`alerts`)](./api-alerts.md)
- [Currency](./api-currency.md)
- [Language detection (`language`)](./api-language.md)
- [License](./api-license.md)
- [Text-to-speech (`tts`)](./api-tts.md)
- [LLM Access (`llm`)](./api-llm.md)
- [yt-dlp downloads (`ytdlp`)](./api-ytdlp.md)
- [Utilities](./api-utilities.md)
