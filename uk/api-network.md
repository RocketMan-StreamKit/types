# network

## Вхідний HTTP — `network.endpoints`

**Потрібно:** `WEB_END_POINTS`

```js
await network.endpoints.create(path, method, callbackName);
```

| Аргумент | Опис |
| --- | --- |
| `path` | Суфікс після `/addon/{addonId}/` |
| `method` | `'GET'` або `'POST'` |
| `callbackName` | Ім'я для `events.On` |

Повертає `{ success: true }` або `{ success: false, message }`.

### Ендпоінт, захищений токеном, для web UI

```js
events.On('onParams', ({ query }) => {
  if (query.token !== data.token) {
    return { success: false, message: 'Unauthorized' };
  }
  return api.config.getParams();
});
```

## Socket.IO — `network.socketEndpoints`

**Потрібно:** `SOCKET_END_POINTS`

```js
await network.socketEndpoints.create(path, callbackName);
events.On(callbackName, payload => { /* see api-events.md */ });

await network.socketEndpoints.emit(path, event, data, socketId?);
```

Підключення клієнта:

```js
// path: /addon/socket.io
// namespace: /addon/{addonId}/{path}
const socket = io('http://localhost:3000/addon/my_addon/live', {
  path: '/addon/socket.io',
});
```

Пропустіть `socketId` в `emit` для broadcast.

## Вихідний HTTP — `network.request`

**Потрібно:** `NETWORK_REQUEST`

| Method | Опис |
| --- | --- |
| `get(url, headers?)` | GET, відповідь як текст |
| `post(url, body, headers?)` | POST JSON (`Content-Type: application/json`) |
| `put(url, body, headers?)` | PUT JSON |
| `delete(url, headers?)` | DELETE |
| `postForm(url, fields, headers?)` | POST `application/x-www-form-urlencoded` |

Обмеження: 5 одночасних, таймаут 10 с, без приватних хостів, перенаправлення не слідуються.

```js
const text = await network.request.get('https://api.example.com/v1/me');
const created = await network.request.post('https://api.example.com/v1/items', { title: 'Hello' });
```

## Вихідний WebSocket — `network.websocket`

**Потрібно:** `NETWORK_WEBSOCKET`

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

`Send` приймає рядок або JSON-серіалізований об'єкт. Максимум 5 одночасних з'єднань.

## Вихідний SignalR — `network.signalr`

**Потрібно:** `NETWORK_SIGNALR`

Глобали `LogLevel` і `HttpTransportType` з `@microsoft/signalr` доступні в пісочниці.

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

`CreateSignalRConnection` перевіряє URL (лише `http://` / `https://`, той самий blocklist хостів, що й HTTP) і повертає `HubConnectionBuilder` з уже викликаним `withUrl`. Повторний `withUrl` і зміна `connection.baseUrl` заборонені. Максимум 5 одночасних запущених з'єднань. Кастомні `httpClient` і об'єкти транспорту не дозволені.
