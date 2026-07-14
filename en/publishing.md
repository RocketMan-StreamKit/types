# Publishing and releases

## Publishing to the catalog

Third-party developers publish addons on the [RocketMan Addons](https://rocketman-streams.com/#addons) page.

1. Host your addon in a **public** GitHub repository and publish at least one [GitHub Release](#github-release-layout) with `main.zip`, `manifest.json`, and `logo.png` / `logo.svg`.
2. Sign in on the site and open the **Addons** section.
3. Add your addon by entering the GitHub repository address — `https://github.com/ORG/REPO` or `ORG/REPO`.
4. The site reads the **latest GitHub release** and parses addon metadata automatically from `manifest.json` (name, version, type, description, permissions, and so on) and from the logo asset.
5. After moderation and approval, the addon appears in the public catalog and in StreamKit+ settings.

To ship an update, publish a new GitHub release and run a [catalog sync](#catalog-file-caching) — on the site (*Update version*) or via the [sync API](#automatic-catalog-version-sync).

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

The StreamKit+ catalog backend and the app's update checker read the **`manifest.json` release asset** to determine the published version. Keep `version` in that file in sync with the release tag (for example tag `v1.2.0` → `"version": "1.2.0"`). The release asset must be **byte-identical** to `manifest.json` inside `main.zip` (required for [catalog sync](./publishing.md#catalog-sync-validation)).

## Checklist before publishing

1. `manifest.json` `id` equals `ORG/REPO` for your GitHub repository.
2. `version` matches the GitHub release tag.
3. Release contains `main.zip`, `manifest.json`, and `logo.png` / `logo.svg`.
4. `main.zip` includes every file required to run the addon (including `manifest.json` and the icon).
5. Permissions in the manifest match what the addon actually uses.
6. After publishing the release, trigger a **catalog sync** (on the site or via the [sync API](#automatic-catalog-version-sync)) so users receive the new files.

## Catalog file caching

When your addon is added to the catalog, the site downloads the release assets (`manifest.json`, `logo.png` / `logo.svg`, `main.zip`) and **caches them permanently**.

- Cached files are **served to all users** and are **not** re-fetched automatically.
- They are refreshed **only** when a version sync runs — manually on the site (*Update version*) or through the [sync API](#automatic-catalog-version-sync).

Once your addon has been moderated and approved, the files users download stay exactly as reviewed. Changing files in a GitHub release **without** running a sync does not change what users get. This is intentional and protects users from silent post-moderation file swaps.

To ship a new version, publish a new GitHub release **and** trigger a sync.

## Catalog sync validation

When adding or syncing an addon, the catalog checks that:

1. The release contains `manifest.json`, `logo.png` (or `logo.svg`), and `main.zip`.
2. The `id` in `manifest.json` matches your repository using the same rules as [manifest.json](./manifest.md): **`ORG/REPO`** for third-party repositories, or **`REPO`** (repository name only) for addons in the `RocketMan-StreamKit` organization.
3. The `manifest.json` release asset is **identical** to the `manifest.json` inside `main.zip`.

If any check fails, the add or sync is rejected with an explanatory error.

## Automatic catalog version sync

Use the developer sync API to refresh the catalog after each GitHub release, without opening the site and clicking *Update version* manually.

> **Base URL in examples:** `https://rocketman-streams.com`. Use the domain of the site where your addon is published.

### Developer token

1. Open the **Addons** page on the site.
2. Click the **API key** button at the top.
3. Copy your developer token from the modal.

You can regenerate the token from the same modal (with confirmation). Regenerating **immediately invalidates** the old token — update it everywhere it is used.

**Keep the token secret.** Anyone with it can trigger version updates for your addons. Store it as a CI/CD secret (for example a GitHub Actions secret named `ROCKETMAN_ADDON_TOKEN`) and never commit it to your repository.

### Sync API

```
POST https://rocketman-streams.com/api/extensions/dev/sync
Content-Type: application/json
```

| Parameter | Required | Description |
| --- | --- | --- |
| `repo` | yes | Addon repository in `OWNER/NAME` format (same as `github.repository` in GitHub Actions). Must match an addon **you own** in the catalog. |
| `token` | yes | Your developer token from the **API key** modal. |

Parameters may also be sent as `application/x-www-form-urlencoded` or URL query parameters; JSON is recommended.

On success the endpoint re-reads the latest GitHub release, re-downloads and refreshes the cached files, and updates the stored version and metadata when the release changed.

**Example:**

```bash
curl -X POST "https://rocketman-streams.com/api/extensions/dev/sync" \
  -H "Content-Type: application/json" \
  -d '{"repo":"OWNER/REPO","token":"YOUR_TOKEN"}'
```

**Success response:**

```json
{
  "status": true,
  "repo": "OWNER/REPO",
  "updated": true,
  "version": "1.2.3"
}
```

- `updated` — `true` if version or metadata changed compared to what was stored; `false` if the release was re-read but nothing changed.
- `version` — the version after the sync.

**Error responses** (HTTP status plus `error` field):

| HTTP | `error` | Meaning |
| --- | --- | --- |
| 400 | `token_required` | The `token` parameter is missing. |
| 400 | `invalid_repo` | The `repo` parameter is missing or malformed. |
| 401 | `invalid_token` | No addon developer owns this token. |
| 403 | `not_owner` | The addon exists but is not owned by the token's owner. |
| 404 | `addon_not_found` | No catalog addon matches `repo`. |
| 429 | `rate_limited_token` | Too many requests for this token. See `retryAfter` (seconds). |
| 429 | `rate_limited_repo` | Too many requests for this repository. See `retryAfter` (seconds). |
| 502 | `sync_failed` | The GitHub release could not be fetched or parsed (see `message`). |
| 500 | `internal_error` | Unexpected server error. |

### Rate limits

- **1 request per 30 seconds** per developer token.
- **1 request per 60 seconds** per repository.

When rate-limited, the response includes `retryAfter` (in seconds).

### GitHub Actions example

This workflow runs on every published release and notifies the catalog via **GitHub OIDC** (no repository secrets). Save it as `.github/workflows/rocketman-sync.yml` in your addon repository. The workflow needs `permissions: id-token: write`; your GitHub org/repo must be linked to your developer account on the RocketMan site.

```yaml
name: Sync addon to RocketMan catalog

on:
  release:
    types: [published]
  workflow_call:
  workflow_dispatch:

permissions:
  id-token: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Get OIDC token
        id: oidc
        run: |
          TOKEN=$(curl -s \
            -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "${ACTIONS_ID_TOKEN_REQUEST_URL}&audience=rocketman-streamkit_plus-extension" \
            | jq -r '.value')

          echo "token=$TOKEN" >> "$GITHUB_OUTPUT"

      - name: Notify catalog about the new release
        run: |
          curl -sS -X POST "https://rocketman-streams.com/api/extensions/dev/sync" \
            -H "Authorization: Bearer ${{ steps.oidc.outputs.token }}"
```

## Related reading

- [manifest.json](./manifest.md) — field reference
- [Getting started](./getting-started.md) — project layout during development
