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

Active `alerts.started()` counts for this addon are cleared by the host on disable, uninstall, reload, and crash-stop — other addons receive `{ active: false }` via `alerts.onChange` when that clears the last in-flight alert. See [Alerts](./api-alerts.md).

## Graceful stop — `addon:prepare-stop`

Before StreamKit+ kills the addon worker, the main process fires the built-in event `addon:prepare-stop` and waits for the handler to finish (with a short timeout, about **10 seconds**). Use it to finish remote cleanup that must run while the OAuth token / network stack are still available.

**When it fires**

- The user **disables** the addon in settings
- The addon is **uninstalled** (or deactivated as a dependency of an uninstall)
- The **application is quitting** (all integration workers receive prepare-stop before kill)

**When it does not fire**

- Worker **crash** / disconnect (no time for a graceful handler)
- Sudden process kill where the main process cannot await workers

**Handler contract**

```js
events.On('addon:prepare-stop', async () => {
  // e.g. pause or disable remote resources (Twitch rewards, webhooks, …)
  await cleanupRemoteState();
  return { success: true };
});
```

| Detail | Value |
| --- | --- |
| Payload | `{}` (empty object) |
| Permission | None |
| Timeout | ~10s — keep work short; unfinished work is abandoned and the worker is killed |
| Optional | Omitting the handler is fine; other addons are unaffected |

Do **not** rely on this event for data that must survive forever — persist important state with `storage` / `api.config` during normal operation. Prefer `addon:prepare-stop` for best-effort remote teardown (pause rewards, close sessions, revoke short-lived hooks).

See also [events](./api-events.md#built-in-main--addon-events).

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
3. In addon settings the addon stays **enabled** but shows a **Stopped after errors** badge (with a tooltip that a manual restart is required).
4. After an **app** restart the crash-stop flag is cleared and the addon is started again automatically.

**How to recover**

During the same session the user must **restart the addon manually** from settings (↻ button) or disable and re-enable it. That clears the crash-stop flag and starts a fresh worker with a reset crash counter. Restarting the whole app also clears the flag and starts enabled addons again.

**Implications for addon authors**

- Do not rely on crashing the worker as a recovery mechanism — after three crashes the addon stays offline until the user restarts it (or restarts the app).
- Keep the initial `index.js` load phase short; defer heavy or risky work to async callbacks after load.
- Use `try/catch` in long-running code paths; unhandled rejections or throws outside sandbox-wrapped handlers can still take down the worker.
- `api.restart()` only works while the addon is running; it cannot restart an addon that was stopped by crash-loop protection.
- `api.getProcessStats()` reports this worker's CPU, RAM, and PID. CPU percent needs at least two calls ~1 s apart; use it to detect leaks or trigger `api.restart()` when limits are exceeded.

## Static web routes

When `manifest.web` is set and `WEB_CONTENT` (or `ROOT`) is granted, the main process registers static routes:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/
```

The root serves the `web` file; paths in `web_contents` are served under the same prefix. Routes are removed on disable, restart, or uninstall.

## Missing files

If `manifest.json` or `index.js` is missing at the recorded install path, the app treats the addon as broken: settings show an error banner, settings cannot be opened, and only uninstall is available. Overlay/timer triggers referencing that addon are cleaned up on startup.
