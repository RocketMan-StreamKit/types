# Utilities

## `api.getProcessStats`

Returns CPU and RAM usage of **this addon worker process**. No extra permissions are required.

| Field | Type | Notes |
| --- | --- | --- |
| `cpuPercent` | `number` | CPU load since the previous call (first call returns `0`) |
| `ramMb` | `number` | Resident memory in megabytes |
| `pid` | `number` | OS process id of this addon worker |

Use for self-monitoring, optimization, or automatic recovery (for example restart when memory exceeds a threshold). Pair with `api.restart()` or internal cache cleanup.

```js
const MAX_RAM_MB = 512;

setInterval(() => {
  const stats = api.getProcessStats();
  console.log('Worker usage', stats);

  if (stats.ramMb > MAX_RAM_MB) {
    console.warn('Memory limit exceeded, restarting worker');
    api.restart();
  }
}, 10_000);
```

The same metrics are shown in addon settings lists in StreamKit+ (PID in the UI only when developer mode is enabled).

## `isDeveloperMode`

`true` when the app runs unpackaged (dev build) or the user enabled **Developer mode** in settings. Use for extra logging or debug endpoints; keep normal behaviour when `false`. Changing the setting requires app/worker restart.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
```

## `ADDON_TMP_DIR`

Absolute path to this addon's scratch directory under the OS temp folder:

`{os.tmpdir()}/StreamKitPlusAddons/{addonId}`

Created automatically when the worker starts. If `addonId` contains `/` (for example `ORG/REPO`), nested folders are used. Paths inside this directory do **not** need `files.requestAccess` consent. Use with `files.*` (`FILE_ACCESS`) or `ytdlp.downloadFile`. Endpoint handlers can return `{ file: pathInsideTmp }` to stream media to the addon web UI.

```js
const sep = ADDON_TMP_DIR.includes('\\') ? '\\' : '/';
const out = `${ADDON_TMP_DIR}${sep}clip.%(ext)s`;
await ytdlp.downloadFile(url, out, { format: 'ba/bestaudio', extractAudio: true, audioFormat: 'm4a' });
```

## `console`

`log`, `error`, `warn`, `info` — prefixed with `[Addon {id}]`.

## Timers

| API | Notes |
| --- | --- |
| `setTimeout(fn, delay, ...args)` | Delay clamped to max 60 000 ms; errors caught |
| `setInterval(fn, interval, ...args)` | Interval clamped 50–60 000 ms |
| `clearTimeout` / `clearInterval` | Standard |
| `sleep(ms)` | Promise; max 60 000 ms |

## `random`

```js
const n = random.number(1, 6);
const id = random.id();
```

## `crypto`

```js
const { verifier, challenge } = crypto.createPkce();

const ok = crypto.verifyRsaSha256(publicKeyPem, message, signatureBase64);
```

## `URL` and `URLSearchParams`

Sandbox-safe implementations for parsing and building URLs inside addon code.

## `require(name)`

Available only with `ROOT` permission. Throws otherwise.
