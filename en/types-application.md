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

The application shell loads your HTML in an iframe. The main process builds the iframe URL:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}&bg={hex}&dataTheme={themeId}
```

| Query param | Description |
| --- | --- |
| `token` | Per-install access token (`data.token` in the worker). Validate on private API routes. |
| `bg` | Active theme background color (`--bg`), URL-encoded hex (e.g. `%230e0e10` for `#0e0e10`). |
| `dataTheme` | Value for the `data-theme` attribute on `<html>`: `light`, `dark`, a custom theme id (`custom-…`), or a built-in preset id (`preset-…`). |

Same static route as widgets. Window id: `addon-app:{addonId}`.

You do **not** need to read these query params yourself when you link `/data/styles.css` — StreamKit+ injects a small bootstrap script automatically (see [Themes](#themes) below).

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

A dynamic `<link>` or any reference to `/data/styles.css` in the HTML source (including inline scripts) also counts.

Use the app's web server port if it differs (`WEB_SERVER_PORT` defaults to `3000`).

Notes:

- No addon permission is required — the route is served by the main app, not from the addon install folder.
- `/data/styles.css` includes built-in light/dark themes (in the base bundle), **all** shipped preset schemes (`preset-…`), and the user's saved custom themes. It is regenerated when custom themes change in **Settings → Interface**.
- Theme variables use the same names as the main window (`--bg`, `--text`, `--btn-primary`, …). Overrides apply per `data-theme` value on `<html>` (same attribute as the main app).
- Your own CSS can override or extend these rules after the stylesheet link.

## Themes

When StreamKit+ serves an **application** addon HTML page that references `/data/styles.css`, it injects a small inline bootstrap script before `</head>`. The script:

1. On load — reads `dataTheme` and `bg` from the iframe URL and sets `document.documentElement` (`data-theme`, `background`, `color-scheme`).
2. On theme change — listens for `postMessage` from the application shell window (no iframe reload).
3. On URL change — re-reads query params if `location.search` changes.

You do not need to copy this script into your addon — link `/data/styles.css` and the shell handles theme sync.

### `data-theme` values

| Value | Meaning |
| --- | --- |
| `light` | Built-in light theme |
| `dark` | Built-in dark theme |
| `custom-…` | User-created custom scheme id |
| `preset-…` | Built-in preset scheme id (e.g. `preset-twitch`) |

When the user selects **System** in settings, `dataTheme` resolves to `light` or `dark` based on the OS preference.

### Live updates (`postMessage`)

When the user changes the theme (or the OS switches light/dark while **System** is selected), the application shell posts into the iframe:

```json
{
  "type": "streamkit:theme",
  "dataTheme": "preset-twitch",
  "bg": "#0e0e10"
}
```

The injected bootstrap script applies this payload automatically. To handle theme changes in your own JavaScript (optional):

```js
window.addEventListener('message', (event) => {
  const payload = event.data;
  if (!payload || payload.type !== 'streamkit:theme') return;
  console.log('Theme changed:', payload.dataTheme, payload.bg);
});
```

### Manual theme handling (advanced)

If you **do not** link `/data/styles.css`, no bootstrap script is injected and theme query params are not applied automatically. You can still read them on load:

```js
const params = new URLSearchParams(location.search);
const dataTheme = params.get('dataTheme');
const bg = params.get('bg');
if (dataTheme) document.documentElement.setAttribute('data-theme', dataTheme);
if (bg) document.documentElement.style.background = bg;
```

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
