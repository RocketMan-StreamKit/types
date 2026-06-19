# status, notify, ui

## status

**Требуется:** `STATUS`

```js
status.Update({
  current: 'online', // offline | connecting | online | error
  message: { en: 'Connected', ru: 'Подключено', uk: 'Підключено' },
});

status.OnClick(() => {
  api.openUrl('https://example.com/dashboard');
});
```

Кликабельность отражается в строке состояния главного окна, когда зарегистрирован `OnClick`.

## notify

**Требуется:** `NOTIFY`

```js
notify.Send({
  id: `${data.id}_status`,
  type: 'success', // success | info | warning | error
  title: { en: 'My Addon' },
  message: { en: 'Connection restored' },
  temp: true, // сбрасывается при следующем запуске приложения
});
```

Если задан `id`, предыдущее уведомление с тем же id заменяется. Используйте стабильные id для обновлений состояния соединения.

```js
notify.Remove(`${data.id}_status`);
```

Удаляет уведомление только если его создал этот аддон. Уведомления других аддонов и основного процесса игнорируются.

## ui.auth

Страницы результата OAuth на локальном веб-сервере:

```js
ui.auth.generateSuccess('Account linked');
ui.auth.generateFail('Access denied');
```

Возврат из HTTP-обработчика:

```js
return { redirect: ui.auth.generateSuccess() };
```

Опциональный query-параметр `message` показывается на странице.
