# Схема налаштувань (`GenerateConfig`)

Викликайте `GenerateConfig([...])` під час завантаження аддона, щоб оголосити поля налаштувань. Схеми зберігаються в конфігурації застосунку; значення — у `addonsParams[id]`. Значення за замовчуванням зливаються під час реєстрації та при відкритті сторінки налаштувань.

## Структура поля

Кожне поле має:

| Властивість | Опис |
| --- | --- |
| `key` | Ключ зберігання в об'єкті params |
| `type` | `text`, `textarea`, `hidden`, `color`, `number`, `boolean`, `array`, `object`, `select`, `choice`, `button`, `triggers`, `folder`, `file`, `info`, `spoiler`, `page` |
| `default` | Початкове значення |
| `editor` | Якщо присутній — поле з'являється в UI налаштувань з міткою, валідацією тощо |
| `options` | Для `select` / `choice`: `{ value, label?, description? }[]` |
| `visibleWhen` | Необов'язково `{ key, equals }` — сховати поле, поки `params[key] !== equals` |
| `items` | Для `array`: `'text'` або `'number'`. Для `spoiler` / `page`: вкладені записи схеми (той самий формат, що й аргументи `GenerateConfig`; рекурсія дозволена) |
| `fields` | Для `object`: вкладені поля |
| `event` | Для `button`: ім'я події при натисканні. Для `triggers`: ім'я події, коли збережене правило збігається з dashboard-подією (обробляйте через `events.On`) |
| `types` | Для `triggers`: необов'язковий білий список типів (`donation`, `subscribe`, `subgift`, `follow`, `custom`) |
| `excludeTypes` | Для `triggers`: необов'язковий чорний список типів (наприклад `['donation']`) |
| `valueContext` | Для `triggers`: необов'язковий контекст, що додається до викликів `create` динамічних значень (наприклад `{ userInput: true }` — згенерована нагорода Twitch запитає в глядача текст) |

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

- **text** — рядок (однорядковий ввід)
- **textarea** — рядок (багаторядковий ввід; зберігання та валідація як у `text`)
- **hidden** — рядок (зберігання та валідація як у `text`); в UI маскується з кнопкою «Показати/Приховати» (за замовчуванням тип password)
- **color** — hex-рядок (`#ff1744`)
- **number** — число
- **boolean** — логічне значення
- **array** — масив текстів або чисел (`items: 'text' | 'number'`)
- **object** — вкладений об'єкт з `fields`
- **select** — `options: { value, label? }[]`; `default` має відповідати опції, інакше використовується перша опція
- **choice** — зберігання як у `select`, але UI показує обраний пункт і кнопку **Обрати…** з модалкою (назва, опис, кнопка вибору)
- **button** — значення не зберігається; при натисканні викликає `events.On(event, …)` (обмежено оголошеними іменами подій)
- **triggers** — масив правил dashboard-тригерів (аддон-джерело, тип, key, value). Рендериться окремим блоком налаштувань з кнопками додати/видалити, як у оверлеїв і ігор. Обов'язковий `event` викликається в цього аддона, коли правило збіглося з `dashboard.addRecord` (`{ key, trigger, record, user }`). Необов'язкові `types` / `excludeTypes` фільтрують види подій. Приклад: показати всі зареєстровані тригери крім донатів — `excludeTypes: ['donation']`.
- **spoiler** — значення не зберігається; кнопка розкриває/ховає вкладені `items` на місці (ті самі записи схеми, що й у `GenerateConfig`, включно з рядками та вкладеними spoiler/page)
- **page** — значення не зберігається; кнопка відкриває окремий екран з `items` і кнопкою «Назад» (рекурсія дозволена)

Поля всередині `spoiler` / `page` зберігаються на **рівні батьківських params** (ключі-сусіди контейнера), а не під ключем контейнера.

Необов'язковий `visibleWhen: { key, equals }` ховає поле в UI, поки значення сусіднього параметра не збіжиться.

## Приклад spoiler / page

```js
{
  key: 'advanced',
  type: 'spoiler',
  editor: {
    label: { en: 'Advanced', ru: 'Дополнительно', uk: 'Додатково' },
    description: { en: 'Optional tuning', uk: 'Додаткові параметри' },
  },
  items: [
    {
      key: 'debug',
      type: 'boolean',
      default: false,
      editor: { label: { en: 'Debug logging', uk: 'Налагоджувальний лог' } },
    },
    {
      key: 'network',
      type: 'page',
      editor: { label: { en: 'Network', uk: 'Мережа' } },
      items: [
        {
          key: 'timeout_ms',
          type: 'number',
          default: 5000,
          editor: { label: { en: 'Timeout (ms)', uk: 'Таймаут (мс)' } },
        },
      ],
    },
  ],
},
```

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

## Приклад поля `triggers`

Поле `triggers` потрібне, коли аддон має реагувати на dashboard-події, які користувач обирає в налаштуваннях (нагороди Twitch, фоллоу тощо) — окремо від вбудованої обробки донатів.

```js
GenerateConfig([
  {
    key: 'order_triggers',
    type: 'triggers',
    event: 'onOrderTrigger',
    excludeTypes: ['donation'],
    editor: {
      label: { en: 'Extra triggers', ru: 'Дополнительные триггеры', uk: 'Додаткові тригери' },
      description: {
        en: 'Channel point rewards and other events besides donations',
        uk: 'Нагороди за бали каналу та інші події, крім донатів',
      },
    },
  },
]);

events.On('onOrderTrigger', ({ key, trigger, record, user }) => {
  console.log(key, trigger.type, trigger.value, record, user);
});
```

`key` — ключ поля схеми (`order_triggers`). `trigger` — вхідна dashboard-подія. Прив’язки потрапляють у `triggers.getApplied()` як `addon` / `addonInput`.

## Читання params

```js
const params = await api.config.getParams();
await api.config.updateParams({ last_sync: Date.now() });
```

Ліміт розміру: **10 000** байт JSON на аддон, якщо не надано `INCREASE_CONFIG_SIZE` (**1 000 000** байт).
