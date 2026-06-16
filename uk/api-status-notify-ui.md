# status, notify, ui

## status

**Потрібно:** `STATUS`

```js
status.Update({
  current: 'online', // offline | connecting | online | error
  message: { en: 'Connected', ru: 'Подключено', uk: 'Підключено' },
});

status.OnClick(() => {
  api.openUrl('https://example.com/dashboard');
});
```

Клікабельність відображається в рядку стану головного вікна, коли зареєстровано `OnClick`.

## notify

**Потрібно:** `NOTIFY`

```js
notify.Send({
  id: `${data.id}_status`,
  type: 'success', // success | info | warning | error
  title: { en: 'My Addon' },
  message: { en: 'Connection restored' },
  temp: true, // cleared on next app start
});
```

Коли встановлено `id`, попереднє сповіщення з тим самим id замінюється. Використовуйте стабільні id для оновлень стану з'єднання.

```js
notify.Remove(`${data.id}_status`);
```

Видаляє сповіщення лише якщо його створив цей аддон. Сповіщення інших аддонів і головного процесу ігноруються.

## ui.auth

Сторінки результату OAuth на локальному web-сервері:

```js
ui.auth.generateSuccess('Account linked');
ui.auth.generateFail('Access denied');
```

Поверніть з HTTP-обробника:

```js
return { redirect: ui.auth.generateSuccess() };
```

Необов'язковий query-параметр `message` показується на сторінці.
