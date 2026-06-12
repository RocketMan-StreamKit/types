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
