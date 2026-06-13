# Lifecycle

## Startup sequence

1. Main process starts the integrations worker (`WorkerMain('integrations')`) and waits for `loadComplete`.
2. Main sends `executeCode` with `{ integration: { id, name, permissions, … }, codePath }`.
3. Worker builds `GenerateContext()`, then runs addon `index.js` inside `vm.runInContext` (initial execution timeout: **5 seconds**).
4. Addon code registers endpoints, config schema, and event handlers during this load phase.

Long-running work must continue **after** load using `setTimeout`, `setInterval`, `sleep`, or async handlers — not blocking synchronous code during the initial VM run.

## When the worker restarts

The worker restarts when:

- The user disables and re-enables the addon
- The addon is reinstalled or its files change (depending on app behaviour)
- `api.restart()` is called from addon code
- Developer mode toggle (requires full app/worker restart to propagate `isDeveloperMode`)

`storage.Read()` / `storage.Write()` persist across worker restarts within the same install. `api.config` params persist in app config.

## Static web routes

When `manifest.web` is set and `WEB_CONTENT` (or `ROOT`) is granted, the main process registers static routes:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/
```

The root serves the `web` file; paths in `web_contents` are served under the same prefix. Routes are removed on disable, restart, or uninstall.

## Missing files

If `manifest.json` or `index.js` is missing at the recorded install path, the app treats the addon as broken: settings show an error banner, settings cannot be opened, and only uninstall is available. Overlay/timer triggers referencing that addon are cleaned up on startup.
