# Permissions

Permissions are declared in `manifest.json` and fixed for the lifetime of the worker. The user must approve them at install time.

Check at runtime with `permissions.has(AddonsPermission.NAME)`. Denied access logs a warning and returns `false`. `ROOT` bypasses every individual check.

## Permission list

| Permission | Description |
| --- | --- |
| `ROOT` | Grants all capabilities (development only; avoid in production) |
| `INCREASE_CONFIG_SIZE` | Raises per-addon params size limit from 10 KB to 1 MB JSON |
| `CONFIG_READ` | `api.config.getConfig()` — read full application config |
| `CONFIG_WRITE` | `api.config.setConfig()` — write application config |
| `ADDON_CONFIG_READ` | `api.config.getAddonParams(otherAddonId)` — read another addon's params (sensitive; highlighted in UI) |
| `NETWORK_REQUEST` | Outbound HTTP: `network.request.get/post/put/delete/postForm` |
| `NETWORK_WEBSOCKET` | Outbound WebSocket: `network.websocket.connect` |
| `WEB_END_POINTS` | Inbound HTTP routes: `network.endpoints.create` |
| `SOCKET_END_POINTS` | Socket.IO namespaces: `network.socketEndpoints` |
| `WEB_CONTENT` | Serve `manifest.web` and `web_contents` at `/addon_static/{id}/` |
| `STATUS` | Status bar: `status.Update`, `status.OnClick` |
| `NOTIFY` | Title-bar notifications: `notify.Send` |
| `DASHBOARD_EVENTS` | Latest-events widget: `dashboard.addRecord`, `registerTriggers`, … |
| `DASHBOARD_CHAT` | Chat window output and send: `addChatMessage`, `onChatSend`, … |
| `DASHBOARD_CHAT_INCOMING` | Subscribe to chat lines: `onChatMessage` / `offChatMessage` |
| `DASHBOARD_EVENTS_INCOMING` | Subscribe to event records: `onRecord` / `offRecord` |
| `FILE_ACCESS` | Scoped file/folder API: `files.requestAccess`, `readFile`, `writeFile`, … (user approves each path; see [File access](./api-file-access.md)) |
| `TTS` | Text-to-speech: `tts.speak`, `tts.getEngine` (uses user TTS settings; see [TTS](./api-tts.md)) |

## Example

```js
if (!permissions.has(AddonsPermission.NETWORK_REQUEST)) {
  console.warn('Network disabled');
  return;
}

const body = await network.request.get('https://api.example.com/status');
```

```js
if (permissions.has(AddonsPermission.TTS)) {
  await tts.speak('Connection restored');
}
```

## Principle of least privilege

Request only permissions your addon uses. For widget/application pages that only need static hosting, `WEB_CONTENT` alone may be enough. Add `WEB_END_POINTS` when the worker must expose HTTP APIs to the embedded page.
