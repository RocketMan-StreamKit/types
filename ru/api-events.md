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

## Встроенные события (главный процесс → аддон)

Главный процесс может также пушить именованные события в воркер через `events.On`. Это **не** HTTP-колбэки — регистрируйте их так же, без создания эндпоинта.

| Событие | Когда | Payload |
| --- | --- | --- |
| `addon:prepare-stop` | Перед убийством воркера (отключение, удаление или выход из приложения) | `{}` — завершите удалённую очистку; держите обработчик коротким (~5 с таймаут). См. [Жизненный цикл — Корректная остановка](./lifecycle.md#корректная-остановка--addonprepare-stop) |
| `triggers:validate` | Перед сохранением настроек, если могут измениться привязки триггеров | `{ draft }` — верните `{ success: false, message }`, чтобы заблокировать сохранение |
| `triggers:applied-changed` | После сохранения настроек, если привязки триггеров этого аддона изменились | `{ previous, current }` |
| `overlayTriggerValue:{provider}:list\|create\|release` | UI динамических значений триггеров | См. [dashboard triggers](./api-dashboard.md) |
| `gameInputTrigger` | Совпавшее dashboard-событие для game input | `{ actionId, trigger, record, user }` |
| `ytdlp:download-progress` | Пока выполняется `ytdlp.downloadFile` | `{ downloadId, progress }` |
| `fileAccessGranted` / `fileAccessRevoked` | Пользователь выдал или отозвал доступ к файлам | См. [file access](./api-file-access.md) |

### Пример: корректная остановка

```js
events.On('addon:prepare-stop', async () => {
  await pauseRemoteRewards();
  return { success: true };
});
```

## Жизненный цикл

Отпишитесь через `subscription.Destroy()` или при перезапуске воркера.
