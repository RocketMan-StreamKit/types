# addons (RPC)

Addon-to-addon communication separate from `events` (HTTP). Main process routes requests between **enabled** workers.

## `addons.request(targetAddonId, method, params?)`

Call a handler registered in another addon.

```js
const res = await addons.request('other_addon', 'getChannelId', { platform: 'example' });
if (res.success) {
  console.log(res.result);
}
```

Returns `{ success: true, result? }` or `{ success: false, message? }`.

## `addons.onRequest(method, handler)`

Expose a method to other addons.

```js
addons.onRequest('getChannelId', async ({ fromAddonId, params }) => {
  console.log('Request from', fromAddonId);
  return { channelId: '12345' };
});
```

Handler receives `{ fromAddonId, params }` and returns the response payload.

## `addons.offRequest(method)`

Remove a previously registered handler.

## Manifest dependencies

Use `depends_on: ["other_addon"]` in `manifest.json` when your addon requires another addon to be installed and enabled before activation.
