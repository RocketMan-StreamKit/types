# Аддоны платформ

Интеграции платформ подключают внешние стриминговые или донат-сервисы к StreamKit+.

## Манифест

| Поле | Значение |
| --- | --- |
| `type` | `platform.streaming` или `platform.donation` |
| `permissions` | Обычно `NETWORK_REQUEST`, `WEB_END_POINTS`, разрешения dashboard и т.д. |

Оба типа используют один поток установки/включения в **Настройки → Аддоны** (группировка по `type`).

## Типичные задачи воркера

- OAuth или авторизация по API-ключу (токены в `api.config.updateParams`)
- Исходящие вызовы API и WebSocket-соединения
- `dashboard.registerPlatform`, `upsertUser`, `addRecord`, `addChatMessage`
- `status.Update` и `notify.Send` для обратной связи о соединении
- `dashboard.registerTriggers`, когда нужны реакции оверлея
- Входящие вебхуки через `network.endpoints.create`

## Взаимодействие между аддонами

Используйте `addons.request` / `addons.onRequest` и опциональный `depends_on`, когда один аддон платформы отдаёт данные, нужные другим.

## Настройки

Объявляйте пользовательские поля через `GenerateConfig`. Скрывайте секреты (токены), не указывая `editor` для соответствующих ключей.
