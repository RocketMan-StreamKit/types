# Документация для разработчиков аддонов StreamKit+

Интеграционные аддоны расширяют StreamKit+ из изолированного worker-процесса. Код аддона выполняется в VM-песочнице и использует **глобальные** API (`network`, `events`, `api`, …) — импорт SDK внутри worker недоступен.

## Разделы

| Раздел | Описание |
| --- | --- |
| [Начало работы](./getting-started.md) | Архитектура, установка, структура проекта, `@rocketman-streamkit/addon-generator` |
| [Жизненный цикл](./lifecycle.md) | Загрузка, перезапуск аддона и защита от цикла падений |
| [manifest.json](./manifest.md) | Поля манифеста, типы, правила валидации |
| [Публикация и релизы](./publishing.md) | Регистрация в каталоге, структура GitHub-релиза, кэш и синхронизация версии |
| [Настройка TypeScript](./typescript.md) | `tsconfig.json`, типы, сборка |
| [Разрешения](./permissions.md) | Флаги возможностей и подтверждение пользователем |
| [Схема настроек](./settings-schema.md) | `GenerateConfig()` и UI настроек |
| [Локализация](./localization.md) | Многоязычные строки в коде аддона |
| [Безопасность](./security.md) | Ограничения сети, токены, лимиты песочницы |
| [OAuth и секреты](./oauth-and-secrets.md) | Обмен токенов через исходящий HTTP |

### API песочницы

| Раздел | Описание |
| --- | --- |
| [Обзор API](./api-overview.md) | Все глобальные объекты |
| [events](./api-events.md) | Шина событий и привязка HTTP-обработчиков |
| [network](./api-network.md) | HTTP-клиент, эндпоинты, WebSocket, SSE, SignalR, Socket.IO |
| [api.config и storage](./api-config-storage.md) | Параметры, конфиг приложения, файловое хранилище |
| [Доступ к файлам](./api-file-access.md) | Чтение/запись по одобренным пользователем путям (`files` API) |
| [Озвучка текста (`tts`)](./api-tts.md) | Озвучка через TTS-движок пользователя (разрешение `TTS`) |
| [LLM Access (`llm`)](./api-llm.md) | Запросы к нейросетям через профили пользователя (разрешение `LLM`) |
| [Загрузки yt-dlp (`ytdlp`)](./api-ytdlp.md) | Загрузка медиа через встроенный yt-dlp (`FILE_ACCESS` + `NETWORK_REQUEST`) |
| [addons (RPC)](./api-addons-rpc.md) | Запросы и события между аддонами |
| [dashboard](./api-dashboard.md) | Виджет событий, чат, триггеры оверлея |
| [status, notify, ui](./api-status-notify-ui.md) | Строка состояния, онлайн зрителей, уведомления, страницы OAuth |
| [Оповещения (`alerts`)](./api-alerts.md) | Координация показа оповещений между аддонами (без разрешения) |
| [Валюта](./api-currency.md) | Основная валюта пользователя и конвертация сумм |
| [Определение языка (`language`)](./api-language.md) | Определение языка текста через fastText (~176 языков; без разрешения) |
| [Утилиты](./api-utilities.md) | Таймеры, crypto, console, режим разработчика |

### Категории аддонов

| Раздел | Описание |
| --- | --- |
| [Платформенные аддоны](./types-platform.md) | `platform.streaming`, `platform.donation` |
| [Оверлеи](./types-overlay.md) | Эффекты, статический web, простое медиа |
| [Виджеты](./types-widget.md) | Постоянные web-страницы и OBS Browser Source |
| [Приложения](./types-application.md) | Окна внутри основного приложения |
| [Игровые интеграции](./types-game.md) | Внутриигровые эффекты и входные триггеры |

## Типизация

Установите объявления песочницы из npm — версия пакета должна совпадать с целевой версией StreamKit+:

```bash
npm install --save-dev @rocketman-streamkit/types
```

См. [Настройка TypeScript](./typescript.md) — `tsconfig.json` и глобалы (`declare global`, без импортов в коде аддона).
