# StreamKit+ Addon Developer Documentation

Integration addons extend StreamKit+ from an isolated worker process. Addon code runs inside a VM sandbox and uses **global** APIs (`network`, `events`, `api`, …) — no SDK imports inside the worker.

## Categories

| Section | Description |
| --- | --- |
| [Getting started](./getting-started.md) | Architecture, installation, project layout |
| [Lifecycle](./lifecycle.md) | How addons are loaded, restarted, and crash-loop protection |
| [manifest.json](./manifest.md) | Manifest fields, types, and validation rules |
| [Publishing and releases](./publishing.md) | GitHub release layout and addon ID tied to repository |
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
| [network](./api-network.md) | HTTP client, endpoints, WebSocket, Socket.IO |
| [api.config & storage](./api-config-storage.md) | Params, app config, private file storage |
| [addons (RPC)](./api-addons-rpc.md) | Addon-to-addon requests |
| [dashboard](./api-dashboard.md) | Events widget, chat, overlay triggers |
| [status, notify, ui](./api-status-notify-ui.md) | Status bar, notifications, OAuth result pages |
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
