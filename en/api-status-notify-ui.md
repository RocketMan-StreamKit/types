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
