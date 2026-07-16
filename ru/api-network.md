# network

## Входящий HTTP — `network.endpoints`

**Требуется:** `WEB_END_POINTS`

```js
await network.endpoints.create(path, method, callbackName);
```

| Аргумент | Описание |
| --- | --- |
| `path` | Суффикс после `/addon/{addonId}/` |
| `method` | `'GET'` или `'POST'` |
| `callbackName` | Имя для `events.On` |

Возвращает `{ success: true }` или `{ success: false, message }`.

### Эндпоинт с защитой токеном для веб-UI

```js
events.On('onParams', ({ query }) => {
  if (query.token !== data.token) {
    return { success: false, message: 'Unauthorized' };
  }
  return api.config.getParams();
});
```

## Socket.IO — `network.socketEndpoints`

**Требуется:** `SOCKET_END_POINTS`

```js
await network.socketEndpoints.create(path, callbackName);
events.On(callbackName, payload => { /* см. api-events.md */ });

await network.socketEndpoints.emit(path, event, data, socketId?);
```

Подключение клиента:

```js
// path: /addon/socket.io
// namespace: /addon/{addonId}/{path}
const socket = io('http://localhost:3000/addon/my_addon/live', {
  path: '/addon/socket.io',
});
```

Без `socketId` в `emit` — широковещательная рассылка.

## Исходящий HTTP — `network.request`

**Требуется:** `NETWORK_REQUEST`

| Метод | Описание |
| --- | --- |
| `get(url, headers?)` | GET, ответ как текст |
| `post(url, body, headers?)` | POST JSON (`Content-Type: application/json`) |
| `put(url, body, headers?)` | PUT JSON |
| `delete(url, headers?)` | DELETE |
| `postForm(url, fields, headers?)` | POST `application/x-www-form-urlencoded` |

Ограничения: 5 одновременных запросов, таймаут 10 с, без приватных хостов, редиректы не выполняются.

```js
const text = await network.request.get('https://api.example.com/v1/me');
const created = await network.request.post('https://api.example.com/v1/items', { title: 'Hello' });
```

## Исходящий WebSocket — `network.websocket`

**Требуется:** `NETWORK_WEBSOCKET`

```js
const ws = await network.websocket.connect('wss://example.com/socket', {
  headers: { Authorization: 'Bearer token' },
  protocols: 'json',
});

ws.On('open', () => ws.Send({ type: 'hello' }));
ws.On('message', data => console.log(data));
ws.On('close', ({ code, reason }) => console.log(code, reason));
ws.On('error', err => console.error(err));

ws.Close();
ws.Destroy();
```

`Send` принимает строку или JSON-сериализуемый объект. Максимум 5 одновременных соединений.

## Исходящий SignalR — `network.signalr`

**Требуется:** `NETWORK_SIGNALR`

Глобалы `LogLevel` и `HttpTransportType` из `@microsoft/signalr` доступны в песочнице.

```js
const connection = (
  await network.signalr.CreateSignalRConnection(
    'https://example.com/hub?access_token=TOKEN',
    { transport: HttpTransportType.WebSockets }
  )
)
  .withAutomaticReconnect()
  .configureLogging(LogLevel.Information)
  .build();

connection.on('DonationCreated', donation => console.log(donation));
await connection.start();
await connection.invoke('SomeMethod', arg);
await connection.stop();
```

`CreateSignalRConnection` проверяет URL (только `http://` / `https://`, тот же чёрный список хостов, что и HTTP) и возвращает `HubConnectionBuilder` с уже вызванным `withUrl`. Повторный `withUrl` и смена `connection.baseUrl` запрещены. Максимум 5 одновременных запущенных соединений. Кастомные `httpClient` и объекты транспорта не допускаются.
