# Дозволи

Дозволи оголошуються в `manifest.json` і фіксуються на весь час життя worker. Користувач має підтвердити їх під час встановлення.

Перевіряйте під час виконання через `permissions.has(AddonsPermission.NAME)`. Відмовлений доступ записує попередження в лог і повертає `false`. `ROOT` обходить кожну окрему перевірку.

## Список дозволів

| Permission | Опис |
| --- | --- |
| `ROOT` | Надає всі можливості (лише для розробки; уникайте в production) |
| `INCREASE_CONFIG_SIZE` | Підвищує ліміт розміру params на аддон з 10 KB до 1 MB JSON |
| `CONFIG_READ` | `api.config.getConfig()` — читання повної конфігурації застосунку |
| `CONFIG_WRITE` | `api.config.setConfig()` — запис конфігурації застосунку |
| `ADDON_CONFIG_READ` | `api.config.getAddonParams(otherAddonId)` — читання params іншого аддона (чутливо; виділяється в UI) |
| `NETWORK_REQUEST` | Вихідний HTTP: `network.request.get/post/put/delete/postForm` |
| `NETWORK_WEBSOCKET` | Вихідний WebSocket: `network.websocket.connect` |
| `NETWORK_SIGNALR` | Вихідний SignalR: `network.signalr.CreateSignalRConnection` |
| `WEB_END_POINTS` | Вхідні HTTP-маршрути: `network.endpoints.create` |
| `SOCKET_END_POINTS` | Простори імен Socket.IO: `network.socketEndpoints` |
| `WEB_CONTENT` | Обслуговування `manifest.web` і `web_contents` за `/addon_static/{id}/` |
| `STATUS` | Рядок стану: `status.Update`, `status.OnClick` |
| `NOTIFY` | Сповіщення в title bar: `notify.Send` |
| `DASHBOARD_EVENTS` | Віджет останніх подій: `dashboard.addRecord`, `registerTriggers`, `registerAttaches`, … |
| `DASHBOARD_CHAT` | Вивід і ціль відправки у вікні чату: `addChatMessage`, `onChatSend`, … |
| `DASHBOARD_CHAT_SEND` | Вихідна відправка через інші аддони: `dashboard.sendChatMessage` |
| `DASHBOARD_CHAT_INCOMING` | Підписка на рядки чату: `onChatMessage` / `offChatMessage` |
| `DASHBOARD_EVENTS_INCOMING` | Підписка на записи подій: `onRecord` / `offRecord` |
| `FILE_ACCESS` | Доступ до файлів і папок: `files.requestAccess`, `readFile`, `writeFile`, … (користувач схвалює кожен шлях; див. [Доступ до файлів](./api-file-access.md)) |
| `TTS` | Озвучення тексту: `tts.speak`, `tts.getEngine` (використовує налаштування TTS користувача; див. [TTS](./api-tts.md)) |

## Приклад

```js
if (!permissions.has(AddonsPermission.NETWORK_REQUEST)) {
  console.warn('Network disabled');
  return;
}

const body = await network.request.get('https://api.example.com/status');
```

```js
if (permissions.has(AddonsPermission.TTS)) {
  await tts.speak("З'єднання відновлено");
}
```

## Принцип мінімальних привілеїв

Запитуйте лише ті дозволи, які використовує ваш аддон. Для сторінок віджета/застосунку, яким потрібне лише статичне хостування, може вистачити одного `WEB_CONTENT`. Додайте `WEB_END_POINTS`, коли worker має надавати HTTP API для вбудованої сторінки.
