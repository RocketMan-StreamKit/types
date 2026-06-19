# Utilities

## `isDeveloperMode`

`true` when the app runs unpackaged (dev build) or the user enabled **Developer mode** in settings. Use for extra logging or debug endpoints; keep normal behaviour when `false`. Changing the setting requires app/worker restart.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
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
