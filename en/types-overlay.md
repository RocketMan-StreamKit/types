# Overlay addons

Overlays are short-lived effects triggered by rules, timers, or dashboard events.

## Manifest

| Field | Value |
| --- | --- |
| `type` | `overlay` or `overlay.info` |
| `web_type` | `overlay` (when `web` is set) |

`overlay.info` panels render after or during the main effect when `showInfo` is enabled on the parent overlay.

## Variants

### Simple media overlay (no worker)

`manifest.overlay` with `audio` / `video` paths. No `index.js`, no permissions. Playback uses the built-in simple player iframe.

Settings: volume, loop, hide-at-end, duration (may be constrained in manifest).

### Static web overlay (no worker)

`web` + `web_type: "overlay"`, no `index.js`. Requires `WEB_CONTENT` and files listed in `web_contents`.

Static URL: `/addon_static/{id}/`

Runtime settings (volume, duration) may be read from:

```
GET /overlay/settings/{overlayId}?token=
```

When the overlay shell plays the effect, the iframe URL includes `display=app` (in-app overlay window) or `display=web` (common OBS overlay browser source). Opening `/addon_static/{id}/` directly has no `display` param:

```js
const display = new URLSearchParams(window.location.search).get('display');
// null → opened outside the overlay shell
// 'app' → in-app overlay window
// 'web' → common OBS overlay link
```

### Full overlay addon (worker)

`type: "overlay"`, worker `index.js`, `WEB_CONTENT` for custom HTML effect.

Platform addons with `DASHBOARD_EVENTS` can register overlay triggers via `dashboard.registerTriggers()` and pass `trigger` in `addRecord()`.

## Permissions

| Pattern | Permissions |
| --- | --- |
| Static HTML only | `WEB_CONTENT` |
| Worker + HTTP for page | `WEB_CONTENT`, `WEB_END_POINTS`, often `SOCKET_END_POINTS` |

Validate `data.token` on endpoints used only by the overlay web page.
