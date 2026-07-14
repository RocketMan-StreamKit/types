# License (`license`)

Read the application license state from an integration addon. No extra permissions are required.

The raw device license key is **never** exposed. `getInfo()` returns an MD5 hex digest (`keyMd5`) so addons can identify the installation on external services without leaking the key.

## `license.active`

Synchronous flag mirrored in the integration worker. Updated when license validity changes in the main process (same gate as integration triggers).

```js
if (!license.active) {
  console.warn('Integration triggers are disabled');
}
```

## `license.id`

Synchronous **license order ID** (settings → “License ID”). Empty string when no license is active.

```js
if (license.id) {
  console.log('License ID', license.id);
}
```

## `license.keyMd5`

Synchronous MD5 hex digest of the device license key (raw key is never exposed).

```js
console.log('Key fingerprint', license.keyMd5);
```

## `license.getInfo()`

Returns the current license snapshot:

| Field | Type | Notes |
| --- | --- | --- |
| `valid` | `boolean` | Whether the license is active |
| `expiresSeconds` | `number` | Remaining seconds; `-1` = perpetual; `0` = expired |
| `licenseId` | `string` | License order ID (empty when inactive) |
| `keyMd5` | `string` | MD5 hex digest of the device license key |

```js
const info = await license.getInfo();
if (info.success && info.valid) {
  console.log('Order', info.licenseId);
  console.log('Key fingerprint', info.keyMd5);
}
```
