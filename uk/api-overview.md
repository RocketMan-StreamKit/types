# Огляд API

Усі API — **глобальні** у VM integrations worker. Типи постачаються в npm-пакеті `@rocketman-streamkit/types` (`addon.d.ts`).

## Глобальні об'єкти

| Global | Призначення |
| --- | --- |
| `isDeveloperMode` | `true` у dev-збірках або коли користувач увімкнув Developer mode |
| `ADDON_TMP_DIR` | Абсолютний scratch-каталог аддона в temp ОС (`…/StreamKitPlusAddons/{id}`); без згоди для шляхів всередині |
| `LANG` | `current` (`en` / `ru` / `uk`), `onChangeLanguage(cb)` — міст до локалі UI застосунку |
| `permissions` | `list`, `has(permission)` |
| `data` | Метадані екземпляра: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — прив'язка HTTP/Socket.IO callback |
| `network` | HTTP-клієнт, вхідні маршрути, WebSocket, SSE, SignalR, Socket.IO |
| `api` | `openUrl`, `restart`, `getProcessStats`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest`, `emit`, `subscribe`, `getInfo` — RPC і події між аддонами, метадані встановлення |
| `dashboard` | Віджет подій, чат, платформи, тригери оверлею |
| `status` | Рядок стану головного вікна |
| `viewers` | Онлайн глядачів у рядку стану головного вікна (без дозволу) |
| `currency` | Основна валюта користувача та конвертація сум (без дозволу) |
| `language` | Визначення мови тексту через fastText lid.176 (~176 ISO-кодів; без дозволу) |
| `license` | Статус ліцензії та MD5-відбиток ключа пристрою (без дозволу) |
| `notify` | Центр сповіщень у title bar |
| `tts` | Озвучення тексту та інформація про голос (дозвіл `TTS`) |
| `llm` | LLM Access — запити до нейромереж (дозвіл `LLM`) |
| `ytdlp` | Завантаження медіа через вбудований yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
| `storage` | Приватний JSON-файл у папці встановлення |
| `ui.auth` | URL перенаправлення успіху/помилки OAuth |
| `crypto` | `createPkce`, `verifyRsaSha256` |
| `console` | Логування з префіксом |
| `setTimeout` / `setInterval` / `clearTimeout` / `clearInterval` | Таймери з ізоляцією помилок |
| `sleep(ms)` | Promise-затримка |
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
- [Валюта](./api-currency.md)
- [Визначення мови (`language`)](./api-language.md)
- [Ліцензія](./api-license.md)
- [Озвучення тексту (`tts`)](./api-tts.md)
- [LLM Access (`llm`)](./api-llm.md)
- [Завантаження yt-dlp (`ytdlp`)](./api-ytdlp.md)
- [Utilities](./api-utilities.md)
