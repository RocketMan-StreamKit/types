# Security

## Network egress

Outbound HTTP (`network.request`) and WebSocket (`network.websocket`) URLs are validated:

- DNS lookup with **5 minute** cache
- Blocked: `localhost`, `127.0.0.1`, `::1`, RFC1918 private ranges (`192.168.*`, `10.*`, `172.16–172.31.*`)
- HTTP: max **5** concurrent requests, **10 s** timeout, redirects **not** followed
- WebSocket: max **5** concurrent connections, `ws://` / `wss://` only, same host blocklist

There is **no** dedicated sandbox API for a privileged auth server. OAuth token exchange uses normal `network.request.post` to a public HTTPS URL you configure (see [OAuth and secrets](./oauth-and-secrets.md)).

## Inbound HTTP

Addons never bind ports. Inbound routes are registered on the main-process Express server:

- Addon HTTP: `/addon/{id}/{path}`
- Static web: `/addon_static/{id}/`
- Socket.IO path: `/addon/socket.io`, namespace `/addon/{id}/{path}`

## Params and config

Writes to `api.config.updateParams` and `api.config.setConfig` are validated in the main process (size limits, permission gates).

## VM execution

Initial `index.js` execution has a **5 second** timeout. Errors in timers and event handlers are caught and logged; they do not crash the worker.

## Frontend access token (`data.token`)

Each addon instance receives a stable per-device, per-addon-id, per-install-path secret in `data.token`. It is appended to widget/static URLs as `?token=…`.

**Validate** `query.token === data.token` (or equivalent header) on HTTP endpoints meant only for your embedded web page so other local pages cannot call them.

The token changes if the user reinstalls or moves the addon folder.

## `ROOT` permission

`require()` and unrestricted module access are available only with `ROOT`. Do not request `ROOT` in distributed addons.

## Private storage

`storage.Read()` / `Write()` persist JSON in `{installPath}/storage_data.json`. Not encrypted; treat as local-only cache. Removed on uninstall.
