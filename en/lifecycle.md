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

## Crash loop protection

If the worker process **crashes** (uncaught exception, fatal error, abrupt exit), StreamKit+ restarts it automatically — similar to a normal restart.

To avoid an infinite restart loop, the app tracks **consecutive crashes**:

| Rule | Value |
| --- | --- |
| Max consecutive crashes | **3** |
| Stable run to reset the counter | **30 seconds** after `loadComplete` |

**What counts as a crash**

- Worker process exit or disconnect
- Failure detected by the main-process health check

**What does not count**

- Errors inside `setTimeout` / `setInterval` callbacks and `events.On` handlers — they are caught and logged; the worker keeps running (see [Security](./security.md#vm-execution))
- Intentional restart via `api.restart()`, disable/enable, or manual restart from settings

**When the limit is reached**

1. Auto-restart stops; the addon worker is not started again.
2. The user sees an error notification and an error status for the addon.
3. In addon settings the addon stays **enabled** but shows a **Stopped after errors** badge.
4. After an app restart the addon is still not started until the user acts.

**How to recover**

The user must **restart the addon manually** from settings (↻ button) or disable and re-enable it. That clears the crash-stop flag and starts a fresh worker with a reset crash counter.

**Implications for addon authors**

- Do not rely on crashing the worker as a recovery mechanism — after three crashes the addon will stay offline until user action.
- Keep the initial `index.js` load phase short; defer heavy or risky work to async callbacks after load.
- Use `try/catch` in long-running code paths; unhandled rejections or throws outside sandbox-wrapped handlers can still take down the worker.
- `api.restart()` only works while the addon is running; it cannot restart an addon that was stopped by crash-loop protection.

## Static web routes

When `manifest.web` is set and `WEB_CONTENT` (or `ROOT`) is granted, the main process registers static routes:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/
```

The root serves the `web` file; paths in `web_contents` are served under the same prefix. Routes are removed on disable, restart, or uninstall.

## Missing files

If `manifest.json` or `index.js` is missing at the recorded install path, the app treats the addon as broken: settings show an error banner, settings cannot be opened, and only uninstall is available. Overlay/timer triggers referencing that addon are cleaned up on startup.
