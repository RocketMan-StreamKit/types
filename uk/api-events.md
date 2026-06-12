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

## Життєвий цикл

Скасуйте реєстрацію через `subscription.Destroy()` або при перезапуску worker.
