# Платформені аддони

Платформені інтеграції з'єднують зовнішні стримінгові або донат-сервіси з StreamKit+.

## Manifest

| Поле | Значення |
| --- | --- |
| `type` | `platform.streaming` або `platform.donation` |
| `permissions` | Зазвичай `NETWORK_REQUEST`, `WEB_END_POINTS`, дозволи dashboard тощо |

Обидва типи використовують той самий потік встановлення/увімкнення в **Settings → Addons** (згруповані за `type`).

## Типові обов'язки worker

- OAuth або авторизація API key (зберігайте токени в `api.config.updateParams`)
- Вихідні API-виклики та WebSocket-з'єднання
- `dashboard.registerPlatform`, `upsertUser`, `addRecord`, `addChatMessage`
- `status.Update` і `notify.Send` для зворотного зв'язку про з'єднання
- `dashboard.registerTriggers`, коли потрібні реакції оверлею
- Вхідні webhook через `network.endpoints.create`

## Співпраця між аддонами

Використовуйте `addons.request` / `addons.onRequest` і необов'язковий `depends_on`, коли один платформений аддон надає дані, потрібні іншим.

## Налаштування

Оголошуйте поля для користувача через `GenerateConfig`. Приховуйте секрети (токени), пропускаючи `editor` для відповідних ключів.
