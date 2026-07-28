# Схема настроек (`GenerateConfig`)

Вызывайте `GenerateConfig([...])` при загрузке аддона, чтобы объявить поля настроек. Схемы хранятся в конфиге приложения; значения — в `addonsParams[id]`. Значения по умолчанию подмешиваются при регистрации и при открытии страницы настроек.

## Структура поля

У каждого поля есть:

| Свойство | Описание |
| --- | --- |
| `key` | Ключ хранения в объекте params |
| `type` | `text`, `textarea`, `hidden`, `color`, `number`, `boolean`, `array`, `object`, `select`, `choice`, `button`, `folder`, `file`, `info`, `spoiler`, `page` |
| `default` | Начальное значение |
| `editor` | Если задан — поле показывается в UI настроек с подписью, валидацией и т.д. |
| `options` | Для `select` / `choice`: `{ value, label?, description? }[]` |
| `visibleWhen` | Опционально `{ key, equals }` — скрыть поле, пока `params[key] !== equals` |
| `items` | Для `array`: `'text'` или `'number'`. Для `spoiler` / `page`: вложенные записи схемы (тот же формат, что у аргументов `GenerateConfig`; рекурсия допускается) |
| `fields` | Для `object`: вложенные поля |
| `event` | Для `button`: имя события при клике (обрабатывается через `events.On`) |

Поля **без** `editor` сохраняются (внутренние счётчики, токены), но не показываются в UI.

## Поля в одной строке

Оберните несколько полей во вложенный массив, чтобы отобразить их в одной строке настроек:

```js
GenerateConfig([
  fieldA,
  [fieldB, fieldC],
  fieldD,
]);
```

## Поддерживаемые типы

- **text** — строка (однострочный ввод)
- **textarea** — строка (многострочный ввод; хранение и валидация как у `text`)
- **hidden** — строка (хранение и валидация как у `text`); в UI маскируется с кнопкой «Показать/Скрыть» (по умолчанию тип password)
- **color** — hex-строка (`#ff1744`)
- **number** — число
- **boolean** — логическое значение
- **array** — массив текста или чисел (`items: 'text' | 'number'`)
- **object** — вложенный объект с `fields`
- **select** — `options: { value, label? }[]`; `default` должен совпадать с опцией, иначе берётся первая
- **choice** — хранение как у `select`, но UI показывает выбранный пункт и кнопку **Выбрать…** с модалкой (название, описание, кнопка выбора)
- **button** — значение не хранится; при клике вызывает `events.On(event, …)` (только для объявленных имён событий)
- **spoiler** — значение не хранится; кнопка раскрывает/скрывает вложенные `items` на месте (те же записи схемы, что у `GenerateConfig`, включая строки и вложенные spoiler/page)
- **page** — значение не хранится; кнопка открывает отдельный экран с `items` и кнопкой «Назад» (рекурсия допускается)

Поля внутри `spoiler` / `page` хранятся на **уровне родительских params** (ключи-соседи контейнера), а не под ключом контейнера.

Опциональный `visibleWhen: { key, equals }` скрывает поле в UI, пока значение соседнего параметра не совпадёт.

## Пример spoiler / page

```js
{
  key: 'advanced',
  type: 'spoiler',
  editor: {
    label: { en: 'Advanced', ru: 'Дополнительно', uk: 'Додатково' },
    description: { en: 'Optional tuning', ru: 'Дополнительные параметры' },
  },
  items: [
    {
      key: 'debug',
      type: 'boolean',
      default: false,
      editor: { label: { en: 'Debug logging', ru: 'Отладочный лог' } },
    },
    {
      key: 'network',
      type: 'page',
      editor: { label: { en: 'Network', ru: 'Сеть' } },
      items: [
        {
          key: 'timeout_ms',
          type: 'number',
          default: 5000,
          editor: { label: { en: 'Timeout (ms)', ru: 'Таймаут (мс)' } },
        },
      ],
    },
  ],
},
```

## Пример

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
  // пользователь нажал «Переподключиться» в настройках
});
```

## Чтение params

```js
const params = await api.config.getParams();
await api.config.updateParams({ last_sync: Date.now() });
```

Лимит размера: **10 000** байт JSON на аддон, если не выдано `INCREASE_CONFIG_SIZE` (**1 000 000** байт).
