# Sound effects

**Settings → Sound effects** is a list of alert sounds you own. They play when a dashboard event matches their triggers — independently of [overlays](./user-overlay.md).

Automatic playback needs an active [license](./user-license.md).

## List

- Create a sound, then open it to edit.
- Toggle a sound off to silence it **without** deleting its trigger rules.
- Preview playback from the list.

## Editor

| Field | What it does |
| --- | --- |
| Name | Label in the UI and in [Latest events](./user-events.md) |
| File or URL | Local audio file or a web URL |
| Volume | 0–100 |
| Triggers | Same event model as overlays (source, type, amount / reward / …) |
| Enabled | Master switch |

Save from the detail bar at the bottom.

## Typical use

1. Add a sound for new followers, another for donations above a threshold.
2. Save, then send a test event or wait for a real one.
3. Confirm a sound chip on the event row if you want to replay it.

Cooperative sync does **not** send sounds to other PCs — only overlay and hotkey effects. See [Cooperative sync](./user-coop-sync.md).
