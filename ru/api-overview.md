# Обзор API

Все API — **глобальные** в VM воркера интеграций. Типы поставляются в npm-пакете `@rocketman-streamkit/types` (`addon.d.ts`).

## Глобальные объекты

| Глобальный объект | Назначение |
| --- | --- |
| `isDeveloperMode` | `true` в dev-сборках или при включённом режиме разработчика |
| `ADDON_TMP_DIR` | Абсолютный scratch-каталог аддона в temp ОС (`…/StreamKitPlusAddons/{id}`); без согласия для путей внутри |
| `LANG` | `current` (`en` / `ru` / `uk`), `onChangeLanguage(cb)` — мост к локали UI приложения |
| `permissions` | `list`, `has(permission)` |
| `data` | Метаданные экземпляра: `id`, `name`, `permissions`, `path`, `token`, … |
| `events` | `On(name, handler)` — привязка HTTP/Socket.IO callback |
| `network` | HTTP-клиент, входящие маршруты, WebSocket, SSE, SignalR, Socket.IO |
| `api` | `openUrl`, `restart`, `getProcessStats`, `config.*` |
| `addons` | `request`, `onRequest`, `offRequest`, `emit`, `subscribe`, `getInfo` — RPC и события между аддонами, метаданные установки |
| `dashboard` | Виджет событий, чат, платформы, триггеры оверлея |
| `status` | Строка состояния главного окна |
| `viewers` | Онлайн зрителей в строке состояния главного окна (без разрешения) |
| `currency` | Основная валюта пользователя и конвертация сумм (без разрешения) |
| `language` | Определение языка текста через fastText lid.176 (~176 ISO-кодов; без разрешения) |
| `license` | Статус лицензии и MD5-отпечаток ключа устройства (без разрешения) |
| `notify` | Центр уведомлений в заголовке |
| `tts` | Озвучка текста (разрешение `TTS`) |
| `llm` | LLM Access — запросы к нейросетям (разрешение `LLM`) |
| `ytdlp` | Загрузка медиа через встроенный yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
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
- [Валюта](./api-currency.md)
- [Определение языка (`language`)](./api-language.md)
- [Лицензия](./api-license.md)
- [Озвучка текста (`tts`)](./api-tts.md)
- [LLM Access (`llm`)](./api-llm.md)
- [Загрузки yt-dlp (`ytdlp`)](./api-ytdlp.md)
- [Утилиты](./api-utilities.md)
