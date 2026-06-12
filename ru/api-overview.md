# Обзор API

Все API — **глобальные** в VM воркера интеграций. Источник истины для типов: `data/addon.d.ts` (генерируется из JSDoc песочницы).

## Глобальные объекты

| Глобальный объект | Назначение |
| --- | --- |
| `isDeveloperMode` | `true` в dev-сборках или при включённом режиме разработчика |
| `permissions` | `list`, `has(permission)` |
| `data` | Метаданные экземпляра: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — привязка HTTP/Socket.IO callback |
| `network` | HTTP-клиент, входящие маршруты, WebSocket, Socket.IO |
| `api` | `openUrl`, `restart`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest` — RPC между аддонами |
| `dashboard` | Виджет событий, чат, платформы, триггеры оверлея |
| `status` | Строка состояния главного окна |
| `notify` | Центр уведомлений в заголовке |
| `storage` | Приватный JSON-файл в папке установки |
| `ui.auth` | URL редиректа успеха/ошибки OAuth |
| `crypto` | `createPkce`, `verifyRsaSha256` |
| `console` | Логирование с префиксом |
| `setTimeout` / `setInterval` / `clearTimeout` / `clearInterval` | Таймеры с ограничениями и изоляцией ошибок |
| `sleep(ms)` | Задержка Promise (макс. 60 с) |
| `random` | `number(min, max)`, `id()` |
| `URL`, `URLSearchParams` | Безопасные для песочницы URL-хелперы |
| `GenerateConfig(schema)` | Регистрация схемы настроек |
| `require(name)` | Node require — **только ROOT** |

## Карта документации

- [events](./api-events.md)
- [network](./api-network.md)
- [api.config и storage](./api-config-storage.md)
- [RPC аддонов](./api-addons-rpc.md)
- [dashboard](./api-dashboard.md)
- [status, notify, ui](./api-status-notify-ui.md)
- [Утилиты](./api-utilities.md)
