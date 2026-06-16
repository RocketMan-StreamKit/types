# manifest.json

The manifest describes the addon identity, category, permissions, and optional web UI.

## Example

```json
{
  "name": {
    "en": "My Addon",
    "ru": "Мой аддон",
    "uk": "Мій аддон"
  },
  "description": {
    "en": "Short description for the settings UI.",
    "ru": "Краткое описание для настроек.",
    "uk": "Короткий опис для налаштувань."
  },
  "id": "MyOrg/my-stream-addon",
  "type": "platform.streaming",
  "version": "1.0.0",
  "author": "Your Name",
  "icon": "logo.png",
  "permissions": ["NETWORK_REQUEST", "WEB_END_POINTS"]
}
```

## Required fields

| Field | Type | Description |
| --- | --- | --- |
| `id` | `string` | Stable addon identifier tied to your GitHub repository (`ORG/REPO`). Used in URLs, `depends_on`, and install records |
| `name` | `{ en, ru?, uk? }` | Display name in settings |
| `description` | `{ en, ru?, uk? }` | Short description in settings |
| `type` | `string` | Addon category (see [Addon categories](#addon-categories)) |
| `version` | `string` | Semantic version string |
| `permissions` | `string[]` | Capability flags (see [Permissions](./permissions.md)) |

The `id` must be unique among installed addons. It is used in URLs (`/addon/{id}/…`, `/addon_static/{id}/…`) and does not have to match the folder name on disk.

Set `id` to **`ORG/REPO`** — the GitHub owner (organization or user) and repository name. Example: repository `https://github.com/MyOrg/my-stream-addon` → `"id": "MyOrg/my-stream-addon"`. See [Publishing and releases](./publishing.md) for release layout and update checks.

## Optional fields

| Field | Type | Description |
| --- | --- | --- |
| `author` | `string` | Author name |
| `icon` | `string` | Icon filename in the addon folder |
| `depends_on` | `string[]` | Addon ids that must be installed **and enabled** before this addon can be enabled. Install without activation is always allowed. |
| `platform` | `string \| string[]` | Restrict install to OS: `win32`, `darwin`, `linux`. Omitted = all platforms. |
| `app_version` | `string` | Minimum StreamKit+ version required (`major.minor.patch`, e.g. `1.0.0`). Omitted = any app version. Install and update are blocked when the running app is older. |
| `web` | `string` | Local HTML filename or external `http(s)` URL |
| `web_type` | `"overlay" \| "widget" \| "application"` | Required when `web` is set; must match `type` rules below |
| `web_contents` | `string[]` | Additional static files served with the page |
| `overlay` | `object` | Simple media overlay: `audio` / `video` file paths (no worker required) |

## `web` and `web_type` rules

| `type` | Required `web_type` |
| --- | --- |
| `overlay` | `overlay` |
| `widget` | `widget` |
| `application` | `application` |

With `WEB_CONTENT` permission, static files are served at `/addon_static/{id}/`.

## Addon categories

| `type` | Purpose |
| --- | --- |
| `platform.streaming` | Streaming platform integration (chat, events, live status) |
| `platform.donation` | Donation platform integration |
| `overlay` | Triggered overlay effects |
| `overlay.info` | Info panel shown with overlay effects |
| `widget` | Persistent web widget (OBS browser source, overlay embed) |
| `application` | Standalone in-app window |
| `game` | Game integration (in-game effects from stream events) |

See category docs: [Platform](./types-platform.md), [Overlay](./types-overlay.md), [Widget](./types-widget.md), [Application](./types-application.md), [Game](./types-game.md).

## Permissions in the manifest

List every capability your addon needs. The user approves them at install time. `ROOT` grants all checks but should not be used in production addons.

Permission display names in the app UI use keys `settings.addons.permission.{PERMISSION_NAME}` in the app's language files — that is an app concern, not something addon authors ship.

## Worker vs workerless

| Pattern | Needs `index.js` | Typical permissions |
| --- | --- | --- |
| Full integration | Yes | As required by your logic |
| Static web only | No | `WEB_CONTENT` |
| Simple media overlay | No | None (media paths in `overlay` block) |
