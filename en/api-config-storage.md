# api.config & storage

## `api.config`

All methods are async (IPC to main process).

| Method | Permission | Description |
| --- | --- | --- |
| `getParams()` | — | This addon's settings blob |
| `updateParams(data)` | — | Replace params (size-limited) |
| `getConfig()` | `CONFIG_READ` | Full application config |
| `setConfig(data)` | `CONFIG_WRITE` | Patch application config |
| `getAddonParams(addonId)` | `ADDON_CONFIG_READ` | Another addon's params |

### Examples

```js
const params = await api.config.getParams();
await api.config.updateParams({ counter: (params.counter || 0) + 1 });

const other = await api.config.getAddonParams('other_addon');
if (other.success) {
  console.log(other.params);
}
```

`updateParams` returns `{ success: true, params }` or `{ success: false, message }`.

## `api.openUrl(url)`

Opens URL in the system default browser. No special permission.

## `api.restart()`

Restarts this addon's worker process.

## `storage`

Private JSON file at `{data.path}/storage_data.json`. **No permission required.** Not shown in settings UI.

| Method | Description |
| --- | --- |
| `Read()` | Parse file or `undefined` if missing |
| `Write(data)` | Replace entire file (merge manually if needed) |
| `Clear()` | Delete file |

```js
const state = storage.Read() ?? { seen: [] };
state.seen.push(eventId);
storage.Write(state);
```

Values must be JSON-serializable. File removed on uninstall.

**Difference from `api.config`:** `storage` is for internal runtime state; `api.config` is user-facing settings (and may appear in the settings UI when fields have `editor`).
