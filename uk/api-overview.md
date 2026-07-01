# Огляд API

Усі API — **глобальні** у VM integrations worker. Типи постачаються в npm-пакеті `@rocketman-streamkit/types` (`addon.d.ts`).

## Глобальні об'єкти

| Global | Призначення |
| --- | --- |
| `isDeveloperMode` | `true` у dev-збірках або коли користувач увімкнув Developer mode |
| `LANG` | `current` (`en` / `ru` / `uk`), `onChangeLanguage(cb)` — міст до локалі UI застосунку |
| `permissions` | `list`, `has(permission)` |
| `data` | Метадані екземпляра: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — прив'язка HTTP/Socket.IO callback |
| `network` | HTTP-клієнт, вхідні маршрути, WebSocket, Socket.IO |
| `api` | `openUrl`, `restart`, `getProcessStats`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest` — RPC між аддонами |
| `dashboard` | Віджет подій, чат, платформи, тригери оверлею |
| `status` | Рядок стану головного вікна |
| `viewers` | Онлайн глядачів у рядку стану головного вікна (без дозволу) |
| `notify` | Центр сповіщень у title bar |
| `tts` | Озвучення тексту (дозвіл `TTS`) |
| `ytdlp` | Завантаження медіа через вбудований yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
| `storage` | Приватний JSON-файл у папці встановлення |
| `ui.auth` | URL перенаправлення успіху/помилки OAuth |
| `crypto` | `createPkce`, `verifyRsaSha256` |
| `console` | Логування з префіксом |
| `setTimeout` / `setInterval` / `clearTimeout` / `clearInterval` | Обмежені таймери з ізоляцією помилок |
| `sleep(ms)` | Promise-затримка (макс. 60 с) |
| `random` | `number(min, max)`, `id()` |
| `URL`, `URLSearchParams` | Безпечні для пісочниці URL helper |
| `GenerateConfig(schema)` | Реєстрація схеми налаштувань |
| `require(name)` | Node require — **лише ROOT** |

## Карта документації

- [events](./api-events.md)
- [network](./api-network.md)
- [api.config & storage](./api-config-storage.md)
- [addons RPC](./api-addons-rpc.md)
- [dashboard](./api-dashboard.md)
- [status, notify, ui](./api-status-notify-ui.md)
- [Озвучення тексту (`tts`)](./api-tts.md)
- [Завантаження yt-dlp (`ytdlp`)](./api-ytdlp.md)
- [Utilities](./api-utilities.md)
