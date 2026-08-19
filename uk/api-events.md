# events

Внутрішня шина подій аддона з'єднує вхідні HTTP- і Socket.IO-обробники з вашим кодом.

## `events.On(name, handler)`

| Аргумент | Опис |
| --- | --- |
| `name` | Має збігатися з `callbackName` з `network.endpoints.create` або `network.socketEndpoints.create` |
| `handler` | Async або sync функція; значення повернення стає тілом HTTP-відповіді (для HTTP-маршрутів) |

HTTP-обробники отримують:

```js
{ query, params, body, headers?, rawBody? }
```

`headers` і `rawBody` доступні на POST-маршрутах (корисно для перевірки підпису webhook).

Повертає підписку `{ id, Destroy() }`.

## Приклад: HTTP webhook

```js
await network.endpoints.create('webhook', 'POST', 'onWebhook');

events.On('onWebhook', ({ body, headers, rawBody }) => {
  console.log(body);
  return { received: true };
});
```

URL клієнта:

```
POST http://localhost:{port}/addon/my_addon/webhook
```

## Приклад: повернення redirect (OAuth)

Обробники можуть повертати об'єкти, які розуміє main process:

```js
return { redirect: ui.auth.generateSuccess() };
```

## Обробники Socket.IO

При використанні `network.socketEndpoints.create` обробник отримує:

```js
{
  type: 'connect' | 'disconnect' | 'event',
  socketId,
  event?,      // for type === 'event'
  data?,       // client payload
}
```

Для `type === 'event'` значення повернення, відмінне від `undefined`, надсилається клієнту як `reply`.

## Вбудовані події (головний процес → аддон)

Головний процес також може пушити іменовані події у worker через `events.On`. Це **не** HTTP-колбеки — реєструйте їх так само, без створення ендпоінта.

| Подія | Коли | Payload |
| --- | --- | --- |
| `addon:prepare-stop` | Перед убивством worker (вимкнення, видалення або вихід із застосунку) | `{}` — завершіть віддалений cleanup; тримайте обробник коротким (~5 с таймаут). Див. [Життєвий цикл — Коректна зупинка](./lifecycle.md#коректна-зупинка--addonprepare-stop) |
| `triggers:validate` | Перед збереженням налаштувань, якщо можуть змінитися прив’язки тригерів | `{ draft }` — поверніть `{ success: false, message }`, щоб заблокувати збереження |
| `triggers:applied-changed` | Після збереження налаштувань, якщо прив’язки тригерів цього аддона змінилися | `{ previous, current }` |
| `overlayTriggerValue:{provider}:list\|create\|release` | UI динамічних значень тригерів | Див. [dashboard triggers](./api-dashboard.md) |
| `gameInputTrigger` | Збіглася dashboard-подія для game input | `{ actionId, trigger, record, user }` |
| `ytdlp:download-progress` | Поки виконується `ytdlp.downloadFile` | `{ downloadId, progress }` |
| `fileAccessGranted` / `fileAccessRevoked` | Користувач надав або відкликав доступ до файлів | Див. [file access](./api-file-access.md) |

### Приклад: коректна зупинка

```js
events.On('addon:prepare-stop', async () => {
  await pauseRemoteRewards();
  return { success: true };
});
```

## Життєвий цикл

Скасуйте реєстрацію через `subscription.Destroy()` або при перезапуску worker.
