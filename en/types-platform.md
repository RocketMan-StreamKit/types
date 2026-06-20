# Platform addons

Platform integrations connect external streaming or donation services to StreamKit+.

## Manifest

| Field | Value |
| --- | --- |
| `type` | `platform.streaming` or `platform.donation` |
| `permissions` | Typically `NETWORK_REQUEST`, `WEB_END_POINTS`, dashboard permissions, etc. |

Both types use the same install/enable flow in **Settings → Addons** (grouped by `type`).

## Typical worker responsibilities

- OAuth or API key auth (store tokens in `api.config.updateParams`)
- Outbound API calls and WebSocket connections
- `dashboard.registerPlatform`, `upsertUser`, `addRecord`, `addChatMessage`
- `status.Update` and `notify.Send` for connection feedback
- `tts.speak` when the addon should voice alerts (requires `TTS`; see [Text-to-speech](./api-tts.md))
- `dashboard.registerTriggers` when overlay reactions are needed
- Inbound webhooks via `network.endpoints.create`

## Addon-to-addon cooperation

Use `addons.request` / `addons.onRequest` and optional `depends_on` when one platform addon exposes data others need.

## Settings

Declare user-facing fields with `GenerateConfig`. Hide secrets (tokens) by omitting `editor` on those keys.
