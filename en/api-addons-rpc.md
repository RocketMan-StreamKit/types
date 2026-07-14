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

## `addons.getInfo(addonIds?)`

Read manifest and enabled status for installed addons. **No extra permission.**

Omit `addonIds` to return every installed addon. Pass an array of ids to query specific addons. Unknown ids are included with `missing: true` and `enabled: false`.

```js
const all = await addons.getInfo();
if (all.success) {
  for (const addon of all.addons) {
    console.log(addon.id, addon.enabled, addon.manifest?.version);
  }
}

const pair = await addons.getInfo(['twitch', 'other_addon']);
```

Each entry: `{ id, enabled, manifest, missing }`. `manifest` is the parsed `manifest.json` or `null` when files are missing.

## Manifest dependencies

Use `depends_on: ["other_addon"]` in `manifest.json` when your addon requires another addon to be installed and enabled before activation.
