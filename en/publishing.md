# Publishing and releases

## Repository and addon ID

Host your addon in a **public GitHub repository**. The manifest `id` must match the repository path on GitHub:

| GitHub repository | manifest `id` |
| --- | --- |
| `https://github.com/MyOrg/my-stream-addon` | `"MyOrg/my-stream-addon"` |
| `https://github.com/jane/widget-panel` | `"jane/widget-panel"` |

Format: **`ORG/REPO`** — GitHub organization or user name, slash, repository name.

The `id` is stable across releases. It is used in:

- HTTP routes (`/addon/{id}/…`, `/addon_static/{id}/…`)
- `depends_on` references between addons
- Update checks (catalog API and GitHub releases)

Do not change the `id` after the first public release unless you treat it as a new addon.

## GitHub release layout

Publish each version as a **GitHub Release**. Attach these files to the release:

| Release asset | Required | Description |
| --- | --- | --- |
| `main.zip` | Yes | Zip archive with **all** addon files |
| `manifest.json` | Yes | Standalone copy of the manifest (also inside `main.zip`) |
| `logo.png` or `logo.svg` | Yes | Icon referenced by manifest `icon` (also inside `main.zip`) |

Recommended contents of `main.zip`:

```
main.zip
├── manifest.json
├── index.js              # when the addon has a worker
├── logo.png              # or logo.svg
├── index.html            # when web UI is used
└── ...                   # other assets (web_contents, media, etc.)
```

The StreamKit+ catalog backend and the app's update checker read the **`manifest.json` release asset** to determine the published version. Keep `version` in that file in sync with the release tag (for example tag `v1.2.0` → `"version": "1.2.0"`).

## Checklist before publishing

1. `manifest.json` `id` equals `ORG/REPO` for your GitHub repository.
2. `version` matches the GitHub release tag.
3. Release contains `main.zip`, `manifest.json`, and `logo.png` / `logo.svg`.
4. `main.zip` includes every file required to run the addon (including `manifest.json` and the icon).
5. Permissions in the manifest match what the addon actually uses.

## Related reading

- [manifest.json](./manifest.md) — field reference
- [Getting started](./getting-started.md) — project layout during development
