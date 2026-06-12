# Widget addons

Widgets are **persistent** web pages (alerts, goals, chat overlays). They are not tied to the overlay trigger queue.

## Manifest

| Field | Value |
| --- | --- |
| `type` | `widget` |
| `web` | HTML entry file |
| `web_type` | `widget` (required) |
| `web_contents` | Additional assets |

## Direct URL

Always available when enabled:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}
```

Shown in settings as **Widget URL** for OBS Browser Source. The `token` query parameter is the addon frontend access token.

## Static widget (no worker)

Only `manifest.json`, icon, HTML, and `web_contents`. Permission: `WEB_CONTENT`.

## Full widget (worker)

Worker exposes HTTP or Socket.IO for live data; web page fetches from `/addon/{id}/…` or Socket.IO namespace.

Protect private endpoints:

```js
if (query.token !== data.token) {
  return { error: 'Unauthorized' };
}
```

## Display in overlay shell

App config `widgets[]` per installed widget:

| Field | Purpose |
| --- | --- |
| `display.app` | Embed in transparent overlay app window |
| `display.web` | Embed in OBS overlay browser source shell |

Direct URL works regardless of display flags.

Live sync: Socket.IO namespace `/overlay`, event `widget:sync` with `{ widgets: [{ widgetId, url }] }`.

Snapshot: `GET /widget/snapshot?display=app|web&token=`

## Settings UI

**Settings → Widgets**: install, enable, `GenerateConfig` fields, display checkboxes, copy widget URL.
