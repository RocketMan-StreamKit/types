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

## Lifecycle

Unregister with `subscription.Destroy()` or when the worker restarts.
