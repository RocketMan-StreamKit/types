# network

## Inbound HTTP — `network.endpoints`

**Requires:** `WEB_END_POINTS`

```js
await network.endpoints.create(path, method, callbackName);
```

| Argument | Description |
| --- | --- |
| `path` | Suffix after `/addon/{addonId}/` |
| `method` | `'GET'` or `'POST'` |
| `callbackName` | Name passed to `events.On` |

Returns `{ success: true }` or `{ success: false, message }`.

### Token-protected endpoint for web UI

```js
events.On('onParams', ({ query }) => {
  if (query.token !== data.token) {
    return { success: false, message: 'Unauthorized' };
  }
  return api.config.getParams();
});
```

## Socket.IO — `network.socketEndpoints`

**Requires:** `SOCKET_END_POINTS`

```js
await network.socketEndpoints.create(path, callbackName);
events.On(callbackName, payload => { /* see api-events.md */ });

await network.socketEndpoints.emit(path, event, data, socketId?);
```

Client connection:

```js
// path: /addon/socket.io
// namespace: /addon/{addonId}/{path}
const socket = io('http://localhost:3000/addon/my_addon/live', {
  path: '/addon/socket.io',
});
```

Omit `socketId` in `emit` to broadcast.

## Outbound HTTP — `network.request`

**Requires:** `NETWORK_REQUEST`

| Method | Description |
| --- | --- |
| `get(url, headers?)` | GET, response as text |
| `post(url, body, headers?)` | POST JSON (`Content-Type: application/json`) |
| `put(url, body, headers?)` | PUT JSON |
| `delete(url, headers?)` | DELETE |
| `postForm(url, fields, headers?)` | POST `application/x-www-form-urlencoded` |

Limits: 5 concurrent, 10 s timeout, no private hosts, redirects not followed.

```js
const text = await network.request.get('https://api.example.com/v1/me');
const created = await network.request.post('https://api.example.com/v1/items', { title: 'Hello' });
```

## Outbound WebSocket — `network.websocket`

**Requires:** `NETWORK_WEBSOCKET`

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

`Send` accepts string or JSON-serializable object. Max 5 concurrent connections.

## Outbound Server-Sent Events — `network.sse`

**Requires:** `NETWORK_SSE`

```js
const sse = await network.sse.connect('https://example.com/events', {
  headers: { Authorization: 'Bearer token' },
  lastEventId: '42',
  autoReconnect: true,
  reconnectInterval: 3000,
});

sse.On('open', () => console.log('stream open'));
sse.On('message', ({ data, event, id }) => console.log(event, data, id));
sse.On('error', err => console.error(err));
sse.On('close', () => console.log('stream closed'));

sse.Close();
sse.Destroy();
```

`message` payload: `{ data: string, event: string, id?: string }` (`event` is `"message"` when the SSE `event:` field is omitted). Optional `lastEventId` is sent as the `Last-Event-ID` header (also updated from received `id:` fields when reconnecting).

| Option | Default | Description |
| --- | --- | --- |
| `autoReconnect` | `false` | When `true`, reconnect after stream end or network error (like browser EventSource). `close` fires only after `Close()` / `Destroy()`. |
| `reconnectInterval` | `3000` | Initial reconnect delay in ms (clamped 50–60000). Overridden by SSE `retry:` fields from the server. |

Max 5 concurrent connections; redirects not followed.

## Outbound SignalR — `network.signalr`

**Requires:** `NETWORK_SIGNALR`

Globals `LogLevel` and `HttpTransportType` from `@microsoft/signalr` are available in the sandbox.

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

`CreateSignalRConnection` validates the URL (`http://` / `https://` only, same host blocklist as HTTP) and returns a `HubConnectionBuilder` with `withUrl` already applied. Calling `withUrl` again or changing `connection.baseUrl` is blocked. Max 5 concurrent started connections. Custom `httpClient` and custom transport objects are not allowed.
