# events

The in-addon event bus connects inbound HTTP and Socket.IO handlers to your code.

## `events.On(name, handler)`

| Argument | Description |
| --- | --- |
| `name` | Must match `callbackName` from `network.endpoints.create` or `network.socketEndpoints.create` |
| `handler` | Async or sync function; return value becomes HTTP response body (for HTTP routes) |

HTTP handlers receive:

```js
{ query, params, body, headers?, rawBody? }
```

`headers` and `rawBody` are available on POST routes (useful for webhook signature verification).

Returns a subscription `{ id, Destroy() }`.

## Example: HTTP webhook

```js
await network.endpoints.create('webhook', 'POST', 'onWebhook');

events.On('onWebhook', ({ body, headers, rawBody }) => {
  console.log(body);
  return { received: true };
});
```

Client URL:

```
POST http://localhost:{port}/addon/my_addon/webhook
```

## Example: return redirect (OAuth)

Handlers may return objects understood by the main process:

```js
return { redirect: ui.auth.generateSuccess() };
```

## Socket.IO handlers

When using `network.socketEndpoints.create`, the handler receives:

```js
{
  type: 'connect' | 'disconnect' | 'event',
  socketId,
  event?,      // for type === 'event'
  data?,       // client payload
}
```

For `type === 'event'`, a non-`undefined` return value is emitted to the client as `reply`.

## Built-in main → addon events

The main process can also push named events into your worker via `events.On`. These are **not** HTTP callbacks — you register them the same way, without creating an endpoint.

| Event | When | Payload |
| --- | --- | --- |
| `addon:prepare-stop` | Before the worker is killed (disable, uninstall, or app quit) | `{}` — finish remote cleanup; keep handlers short (~2.5s timeout). See [Lifecycle — Graceful stop](./lifecycle.md#graceful-stop--addonprepare-stop) |
| `triggers:validate` | Before settings save when trigger bindings may change | `{ draft }` — return `{ success: false, message }` to block save |
| `triggers:applied-changed` | After settings save when this addon’s trigger bindings changed | `{ previous, current }` |
| `overlayTriggerValue:{provider}:list\|create\|release` | Dynamic trigger value UI | See [dashboard triggers](./api-dashboard.md) |
| `gameInputTrigger` | Matching dashboard event for a game input rule | `{ actionId, trigger, record, user }` |
| `ytdlp:download-progress` | While `ytdlp.downloadFile` runs | `{ downloadId, progress }` |
| `fileAccessGranted` / `fileAccessRevoked` | User grants or revokes file access | See [file access](./api-file-access.md) |

### Example: graceful stop

```js
events.On('addon:prepare-stop', async () => {
  await pauseRemoteRewards();
  return { success: true };
});
```

## Lifecycle

Unregister with `subscription.Destroy()` or when the worker restarts.
