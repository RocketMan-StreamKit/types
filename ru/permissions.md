# Разрешения

Разрешения объявляются в `manifest.json` и фиксируются на время жизни воркера. Пользователь должен подтвердить их при установке.

Проверяйте в рантайме через `permissions.has(AddonsPermission.NAME)`. При отказе в доступе пишется предупреждение в лог и возвращается `false`. `ROOT` обходит каждую отдельную проверку.

## Список разрешений

| Разрешение | Описание |
| --- | --- |
| `ROOT` | Все возможности (только для разработки; избегайте в продакшене) |
| `INCREASE_CONFIG_SIZE` | Повышает лимит размера params аддона с 10 КБ до 1 МБ JSON |
| `CONFIG_READ` | `api.config.getConfig()` — чтение полного конфига приложения |
| `CONFIG_WRITE` | `api.config.setConfig()` — запись конфига приложения |
| `ADDON_CONFIG_READ` | `api.config.getAddonParams(otherAddonId)` — чтение params другого аддона (чувствительно; выделяется в UI) |
| `NETWORK_REQUEST` | Исходящий HTTP: `network.request.get/post/put/delete/postForm` |
| `NETWORK_WEBSOCKET` | Исходящий WebSocket: `network.websocket.connect` |
| `WEB_END_POINTS` | Входящие HTTP-маршруты: `network.endpoints.create` |
| `SOCKET_END_POINTS` | Пространства имён Socket.IO: `network.socketEndpoints` |
| `WEB_CONTENT` | Раздача `manifest.web` и `web_contents` по `/addon_static/{id}/` |
| `STATUS` | Строка состояния: `status.Update`, `status.OnClick` |
| `NOTIFY` | Уведомления в заголовке окна: `notify.Send` |
| `DASHBOARD_EVENTS` | Виджет последних событий: `dashboard.addRecord`, `registerTriggers`, `registerAttaches`, … |
| `DASHBOARD_CHAT` | Вывод и цель отправки в окне чата: `addChatMessage`, `onChatSend`, … |
| `DASHBOARD_CHAT_SEND` | Исходящая отправка через другие аддоны: `dashboard.sendChatMessage` |
| `DASHBOARD_CHAT_INCOMING` | Подписка на строки чата: `onChatMessage` / `offChatMessage` |
| `DASHBOARD_EVENTS_INCOMING` | Подписка на записи событий: `onRecord` / `offRecord` |
| `FILE_ACCESS` | Доступ к файлам и папкам: `files.requestAccess`, `readFile`, `writeFile`, … (пользователь одобряет каждый путь; см. [Доступ к файлам](./api-file-access.md)) |
| `TTS` | Озвучка текста: `tts.speak`, `tts.getEngine` (использует настройки TTS пользователя; см. [TTS](./api-tts.md)) |

## Пример

```js
if (!permissions.has(AddonsPermission.NETWORK_REQUEST)) {
  console.warn('Network disabled');
  return;
}

const body = await network.request.get('https://api.example.com/status');
```

```js
if (permissions.has(AddonsPermission.TTS)) {
  await tts.speak('Соединение восстановлено');
}
```

## Принцип минимальных привилегий

Запрашивайте только те разрешения, которые реально использует аддон. Для страниц виджета/приложения, которым нужен только статический хостинг, может хватить одного `WEB_CONTENT`. Добавляйте `WEB_END_POINTS`, когда воркер должен отдавать HTTP API встроенной странице.
