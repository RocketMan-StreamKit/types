# events

Внутренняя шина событий аддона связывает входящие HTTP и Socket.IO с вашим кодом.

## `events.On(name, handler)`

| Аргумент | Описание |
| --- | --- |
| `name` | Должен совпадать с `callbackName` из `network.endpoints.create` или `network.socketEndpoints.create` |
| `handler` | Синхронная или async-функция; возвращаемое значение становится телом HTTP-ответа (для HTTP-маршрутов) |

HTTP-обработчики получают:

```js
{ query, params, body, headers?, rawBody? }
```

`headers` и `rawBody` доступны на POST-маршрутах (удобно для проверки подписи вебхука).

Возвращает подписку `{ id, Destroy() }`.

## Пример: HTTP-вебхук

```js
await network.endpoints.create('webhook', 'POST', 'onWebhook');

events.On('onWebhook', ({ body, headers, rawBody }) => {
  console.log(body);
  return { received: true };
});
```

URL клиента:

```
POST http://localhost:{port}/addon/my_addon/webhook
```

## Пример: редирект (OAuth)

Обработчики могут возвращать объекты, которые понимает главный процесс:

```js
return { redirect: ui.auth.generateSuccess() };
```

## Обработчики Socket.IO

При `network.socketEndpoints.create` обработчик получает:

```js
{
  type: 'connect' | 'disconnect' | 'event',
  socketId,
  event?,      // для type === 'event'
  data?,       // payload клиента
}
```

Для `type === 'event'` ненулевое возвращаемое значение отправляется клиенту как `reply`.

## Жизненный цикл

Отпишитесь через `subscription.Destroy()` или при перезапуске воркера.
