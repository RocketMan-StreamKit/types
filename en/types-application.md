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

## Matching main app styles

StreamKit+ serves a bundled copy of the main UI stylesheet from the local web server. Link it in your application HTML so buttons, inputs, typography, and theme CSS variables match the rest of the app.

| Resource | URL |
| --- | --- |
| Full stylesheet | `http://localhost:{WEB_SERVER_PORT}/data/styles.css` |
| Font files | `http://localhost:{WEB_SERVER_PORT}/data/fonts/…` (referenced by the CSS) |

Add to your HTML entry file **before** addon-specific styles:

```html
<link rel="stylesheet" href="http://localhost:3000/data/styles.css">
```

Use the app's web server port if it differs (`WEB_SERVER_PORT` defaults to `3000`).

Notes:

- No addon permission is required — the route is served by the main app, not from the addon install folder.
- Theme variables use the same names as the main window (`--bg`, `--text`, `--btn-primary`, …). Dark overrides apply when `data-theme="dark"` is set on `<html>` (same attribute as the main app).
- Your own CSS can override or extend these rules after the `<link>` tag.

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
