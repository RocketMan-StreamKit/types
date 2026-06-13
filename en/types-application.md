# Application addons

Applications are web UIs opened from the **main window** in a dedicated sandboxed `BrowserWindow` (no Node integration, no preload).

## Manifest

| Field | Value |
| --- | --- |
| `type` | `application` |
| `web` | HTML entry file |
| `web_type` | `application` (required) |
| `web_contents` | Additional assets |

## Launch URL

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}
```

Same static route as widgets. Window id: `addon-app:{addonId}`.

## Static application (no worker)

`WEB_CONTENT` only — HTML/JS/CSS in the install folder.

## Full application (worker)

Worker provides API endpoints for the page. Validate `query.token === data.token` on private routes (same pattern as widgets).

Typical endpoints:

- `GET params` — settings for the page
- `GET state` — current state
- `POST …` — user actions

## Main window UI

When at least one **enabled** application addon with valid `web` exists, the main window shows **Applications**. User picks an item; main process opens or focuses the sandbox window via IPC.

List updates on install/enable/disable/uninstall.

## Settings UI

**Settings → Applications**: `AddonsCategoryBlock` with `type="application"` — same install/enable/settings flow as other categories.
