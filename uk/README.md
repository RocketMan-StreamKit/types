# Документація для розробників аддонів StreamKit+

Інтеграційні аддони розширюють StreamKit+ з ізольованого worker-процесу. Код аддона виконується у VM-пісочниці та використовує **глобальні** API (`network`, `events`, `api`, …) — імпорт SDK всередині worker недоступний.

## Розділи

| Розділ | Опис |
| --- | --- |
| [Початок роботи](./getting-started.md) | Архітектура, встановлення, структура проєкту |
| [Життєвий цикл](./lifecycle.md) | Завантаження та виконання аддона |
| [manifest.json](./manifest.md) | Поля маніфесту, типи, правила валідації |
| [Налаштування TypeScript](./typescript.md) | `tsconfig.json`, типи, збірка |
| [Дозволи](./permissions.md) | Прапорці можливостей і підтвердження користувачем |
| [Схема налаштувань](./settings-schema.md) | `GenerateConfig()` і UI налаштувань |
| [Локалізація](./localization.md) | Багатомовні рядки в коді аддона |
| [Безпека](./security.md) | Обмеження мережі, токени, ліміти пісочниці |
| [OAuth і секрети](./oauth-and-secrets.md) | Обмін токенів через вихідний HTTP |

### API пісочниці

| Розділ | Опис |
| --- | --- |
| [Огляд API](./api-overview.md) | Усі глобальні об'єкти |
| [events](./api-events.md) | Шина подій і прив'язка HTTP-обробників |
| [network](./api-network.md) | HTTP-клієнт, ендпоінти, WebSocket, Socket.IO |
| [api.config і storage](./api-config-storage.md) | Параметри, конфіг застосунку, файлове сховище |
| [addons (RPC)](./api-addons-rpc.md) | Запити між аддонами |
| [dashboard](./api-dashboard.md) | Віджет подій, чат, тригери оверлею |
| [status, notify, ui](./api-status-notify-ui.md) | Рядок стану, сповіщення, сторінки OAuth |
| [Утиліти](./api-utilities.md) | Таймери, crypto, console, режим розробника |

### Категорії аддонів

| Розділ | Опис |
| --- | --- |
| [Платформені аддони](./types-platform.md) | `platform.streaming`, `platform.donation` |
| [Оверлеї](./types-overlay.md) | Ефекти, статичний web, просте медіа |
| [Віджети](./types-widget.md) | Постійні web-сторінки та OBS Browser Source |
| [Застосунки](./types-application.md) | Вікна всередині основного застосунку |
| [Ігрові інтеграції](./types-game.md) | Внутрішньоігрові ефекти та вхідні тригери |

## Типізація

Після змін API пісочниці перегенеруйте оголошення:

```bash
npm run type:addons
```

Результат: `data/addon.d.ts` (`declare global` — використовуйте в TypeScript аддона без імпортів).
