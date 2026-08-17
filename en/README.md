# StreamKit+ Documentation

Guides for **streamers** (how to use the app) and for **addon developers** (sandbox API, manifest, permissions).

Integration addons extend StreamKit+ from an isolated worker process. Addon code runs inside a VM sandbox and uses **global** APIs (`network`, `events`, `api`, …) — no SDK imports inside the worker.

## For streamers

How to use StreamKit+ as a viewer of the desktop app — not how to write addons.

| Section | Description |
| --- | --- |
| [Getting started](./user-getting-started.md) | First launch, license, what each module is |
| [Main window](./user-main-window.md) | Timer, quick actions, title bar, status |
| [Latest events](./user-events.md) | Live event feed and replay chips |
| [Chat window](./user-chat.md) | Combined chat, sending replies, appearance |
| [Settings](./user-settings.md) | Sidebar, save/reset, Main and Interface |
| [Addons and catalog](./user-addons.md) | Install platforms, permissions, updates |
| [Overlay](./user-overlay.md) | Effects on screen and in OBS |
| [Widgets and applications](./user-widgets.md) | OBS widgets and in-app windows |
| [Donation timer](./user-timer.md) | Countdown files and auto rules |
| [Sound effects](./user-sounds.md) | Alert sounds on events |
| [Hotkeys](./user-hotkeys.md) | Keyboard agent and presets |
| [Cooperative sync](./user-coop-sync.md) | Share overlay and hotkeys with other PCs |
| [Game integrations](./user-games.md) | Viewer actions inside a game |
| [Text to speech](./user-tts.md) | Piper, ElevenLabs, Windows voices |
| [LLM Access](./user-llm.md) | AI profiles for addons |
| [License and updates](./user-license.md) | License, app updates, changelog |
| [Backups](./user-backup.md) | Zip snapshots and restore |

## For addon developers

| Section | Description |
| --- | --- |
| [Getting started](./getting-started.md) | Architecture, installation, project layout, `@rocketman-streamkit/addon-generator` |
| [Lifecycle](./lifecycle.md) | How addons are loaded, restarted, and crash-loop protection |
| [manifest.json](./manifest.md) | Manifest fields, types, and validation rules |
| [Publishing and releases](./publishing.md) | Catalog registration, GitHub release layout, caching, and version sync |
| [TypeScript setup](./typescript.md) | `tsconfig.json`, typings, build output |
| [Permissions](./permissions.md) | Capability flags and user approval |
| [Settings schema](./settings-schema.md) | `GenerateConfig()` and settings UI |
| [Localization](./localization.md) | Per-locale strings in addon code |
| [Security](./security.md) | Network restrictions, tokens, sandbox limits |
| [OAuth and secrets](./oauth-and-secrets.md) | Token exchange via outbound HTTP (no built-in auth server API) |

### Sandbox API

| Section | Description |
| --- | --- |
| [API overview](./api-overview.md) | All globals at a glance |
| [events](./api-events.md) | Event bus and HTTP handler binding |
| [network](./api-network.md) | HTTP client, endpoints, WebSocket, SSE, SignalR, Socket.IO |
| [api.config & storage](./api-config-storage.md) | Params, app config, private file storage |
| [File access](./api-file-access.md) | User-approved read/write to specific paths (`files` API) |
| [Text-to-speech (`tts`)](./api-tts.md) | Play messages via the user's TTS engine (`TTS` permission) |
| [LLM Access (`llm`)](./api-llm.md) | Chat completions via user LLM profiles (`LLM` permission) |
| [yt-dlp downloads (`ytdlp`)](./api-ytdlp.md) | Download media via bundled yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
| [addons (RPC)](./api-addons-rpc.md) | Addon-to-addon requests and events |
| [dashboard](./api-dashboard.md) | Events widget, chat, overlay triggers |
| [status, notify, ui](./api-status-notify-ui.md) | Status bar, viewer count, notifications, OAuth result pages |
| [Alerts (`alerts`)](./api-alerts.md) | Cross-addon alert display coordination (no permission) |
| [Currency](./api-currency.md) | User's primary currency and amount conversion |
| [Language detection (`language`)](./api-language.md) | Detect text language via fastText (~176 languages; no permission) |
| [Utilities](./api-utilities.md) | Timers, crypto, console, developer mode |

### Addon categories

| Section | Description |
| --- | --- |
| [Platform addons](./types-platform.md) | `platform.streaming`, `platform.donation` |
| [Overlay addons](./types-overlay.md) | Effects, static web, simple media |
| [Widget addons](./types-widget.md) | Persistent web pages and OBS sources |
| [Application addons](./types-application.md) | In-app sandboxed windows |
| [Game addons](./types-game.md) | In-game effects and input triggers |

## Typings

Install sandbox declarations from npm — version must match the StreamKit+ release you target:

```bash
npm install --save-dev @rocketman-streamkit/types
```

See [TypeScript setup](./typescript.md) for `tsconfig.json` and globals (`declare global` — no imports in addon code).
