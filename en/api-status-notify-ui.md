# status, notify, ui

## status

**Requires:** `STATUS`

```js
status.Update({
  current: 'online', // offline | connecting | online | error
  message: { en: 'Connected', ru: 'Подключено', uk: 'Підключено' },
});

status.OnClick(() => {
  api.openUrl('https://example.com/dashboard');
});
```

Clickability is reflected in the main window status bar when `OnClick` is registered.

## viewers

**No permission required**

Reports live viewer count for a platform in the main window status bar (left of the platform/version line). Click the block to open display settings.

```js
viewers.Update({
  platform: 'twitch',
  count: 128,
});
```

| Field | Description |
| --- | --- |
| `platform` | Platform identifier. Use the same id as `dashboard.registerPlatform` when possible (e.g. `twitch`, `youtube`). |
| `count` | Non-negative integer viewer count. |

**Behavior:**

- Only addons that call `Update` at least once are included in the UI.
- Disabling the addon removes its viewer data.
- The addon manifest icon is shown next to each platform count; hovering shows the change since the last update.
- The user chooses display mode in the main window: per platform, per platform + total (app icon and sum), or total only. Optional inline delta (+N green / −N red). **Settings → Interface** controls whether zero counts are listed.

Example with platform registration:

```js
await dashboard.registerPlatform({
  id: 'twitch',
  name: { en: 'Twitch', ru: 'Twitch', uk: 'Twitch' },
});

viewers.Update({ platform: 'twitch', count: viewerCount });
```

## notify

**Requires:** `NOTIFY`

```js
notify.Send({
  id: `${data.id}_status`,
  type: 'success', // success | info | warning | error
  title: { en: 'My Addon' },
  message: { en: 'Connection restored' },
  temp: true, // cleared on next app start
});
```

When `id` is set, the previous notification with the same id is replaced. Use stable ids for connection-state updates.

```js
notify.Remove(`${data.id}_status`);
```

Removes a notification only when this addon created it. Notifications from other addons or the main process are ignored.

## settings.notify

Popup notifications shown above the addon settings window (no permission required). They can be sent only while the settings panel for this addon is open.

```js
if (settings.isOpen && !settings.isNotifyBlocked) {
  settings.notify.Send({
    title: { en: 'Settings saved' },
    message: { en: 'Token stored' },
    buttonText: { en: 'Close' },
  });
}
```

More than 5 notifications in 10 seconds blocks popups for 5 seconds. The block state is reflected in `settings.isNotifyBlocked`.

## ui.auth

OAuth result pages on the local web server:

```js
ui.auth.generateSuccess('Account linked');
ui.auth.generateFail('Access denied');
```

Return from an HTTP handler:

```js
return { redirect: ui.auth.generateSuccess() };
```

Optional `message` query parameter is shown on the page.
