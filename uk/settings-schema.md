# Схема налаштувань (`GenerateConfig`)

Викликайте `GenerateConfig([...])` під час завантаження аддона, щоб оголосити поля налаштувань. Схеми зберігаються в конфігурації застосунку; значення — у `addonsParams[id]`. Значення за замовчуванням зливаються під час реєстрації та при відкритті сторінки налаштувань.

## Структура поля

Кожне поле має:

| Властивість | Опис |
| --- | --- |
| `key` | Ключ зберігання в об'єкті params |
| `type` | `text`, `color`, `number`, `boolean`, `array`, `object`, `select`, `button` |
| `default` | Початкове значення |
| `editor` | Якщо присутній — поле з'являється в UI налаштувань з міткою, валідацією тощо |
| `options` | Для `select`: `{ value, label? }[]` |
| `items` | Для `array`: `'text'` або `'number'` |
| `fields` | Для `object`: вкладені поля |
| `event` | Для `button`: ім'я події при натисканні (обробляйте через `events.On`) |

Поля **без** `editor` зберігаються (внутрішні лічильники, токени), але не показуються в UI.

## Поля в одному рядку

Обгорніть кілька полів у вкладений масив, щоб відобразити їх в одному рядку налаштувань:

```js
GenerateConfig([
  fieldA,
  [fieldB, fieldC],
  fieldD,
]);
```

## Підтримувані типи

- **text** — рядок
- **color** — hex-рядок (`#ff1744`)
- **number** — число
- **boolean** — логічне значення
- **array** — масив текстів або чисел (`items: 'text' | 'number'`)
- **object** — вкладений об'єкт з `fields`
- **select** — `options: { value, label? }[]`; `default` має відповідати опції, інакше використовується перша опція
- **button** — значення не зберігається; при натисканні викликає `events.On(event, …)` (обмежено оголошеними іменами подій)

## Приклад

```js
GenerateConfig([
  {
    key: 'api_server',
    type: 'text',
    default: 'https://example.com:2083',
    editor: {
      label: { en: 'API server', ru: 'API сервер', uk: 'API сервер' },
    },
  },
  {
    key: 'theme',
    type: 'select',
    default: 'dark',
    options: [
      { value: 'dark', label: { en: 'Dark', ru: 'Тёмная', uk: 'Темна' } },
      { value: 'light', label: { en: 'Light', ru: 'Светлая', uk: 'Світла' } },
    ],
    editor: { label: { en: 'Theme', ru: 'Тема', uk: 'Тема' } },
  },
  { key: 'last_sync', type: 'number', default: 0 },
  {
    key: 'reconnect',
    type: 'button',
    event: 'onReconnect',
    editor: { label: { en: 'Reconnect', ru: 'Переподключиться', uk: 'Перепідключитися' } },
  },
]);

events.On('onReconnect', async () => {
  // user clicked Reconnect in settings
});
```

## Читання params

```js
const params = await api.config.getParams();
await api.config.updateParams({ last_sync: Date.now() });
```

Ліміт розміру: **10 000** байт JSON на аддон, якщо не надано `INCREASE_CONFIG_SIZE` (**1 000 000** байт).
