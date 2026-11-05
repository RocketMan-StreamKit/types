# Settings schema (`GenerateConfig`)

Call `GenerateConfig([...])` at addon load time to declare settings fields. Schemas are stored in app config; values live in `addonsParams[id]`. Defaults are merged on registration and when the settings page opens.

## Field structure

Each field has:

| Property | Description |
| --- | --- |
| `key` | Storage key in params object |
| `type` | `text`, `textarea`, `hidden`, `color`, `number`, `boolean`, `array`, `object`, `select`, `choice`, `button`, `triggers`, `folder`, `file`, `info`, `spoiler`, `page` |
| `pathPicker` | For `folder` / `file`: `title`, `filters` (file only), `filename` (exact basename), `namePattern` (regex on basename, file only) |
| `default` | Initial value |
| `editor` | If present, field appears in settings UI with label, validation, etc. |
| `options` | For `select` / `choice`: `{ value, label?, description? }[]` |
| `visibleWhen` | Optional `{ key, equals }` — hide the field unless `params[key] === equals` |
| `items` | For `array`: `'text'` or `'number'`. For `spoiler` / `page`: nested schema entries (same shape as `GenerateConfig` arguments; recursion allowed) |
| `fields` | For `object`: nested fields |
| `event` | For `button`: event name fired on click. For `triggers`: event name fired when a saved rule matches a dashboard event (handle with `events.On`) |
| `types` | For `triggers`: optional allow-list of trigger kinds (`donation`, `subscribe`, `subgift`, `follow`, `custom`) |
| `excludeTypes` | For `triggers`: optional deny-list of trigger kinds (for example `['donation']`) |
| `valueContext` | For `triggers`: optional context merged into dynamic value `create` calls (for example `{ userInput: true }` — a generated Twitch reward then asks the viewer for text) |

Fields **without** `editor` are persisted (internal counters, tokens) but not shown in the UI.

## Inline rows

Wrap multiple fields in a nested array to render them on one settings row:

```js
GenerateConfig([
  fieldA,
  [fieldB, fieldC],
  fieldD,
]);
```

## Supported types

- **text** — string (single-line input)
- **textarea** — string (multiline input; same storage/validation as `text`)
- **hidden** — string (same storage and validation as `text`); masked in the UI with a Show/Hide toggle (password field by default)
- **color** — hex string (`#ff1744`)
- **number** — number
- **boolean** — boolean
- **array** — array of text or numbers (`items: 'text' | 'number'`)
- **object** — nested object with `fields`
- **select** — `options: { value, label? }[]`; `default` must match an option or the first option is used
- **choice** — same storage as `select`, but the settings UI shows the selected label/description and a **Choose…** button that opens a modal list (each option: title, description, Select)
- **button** — no stored value; triggers `events.On(event, …)` when clicked (gated to declared event names)
- **triggers** — array of dashboard trigger rules (source addon, type, key, value). Renders as a settings block with add/remove rows, same picker as overlay/game triggers. Required `event` is fired on this addon when a rule matches `dashboard.addRecord` (`{ key, trigger, record, user }`). Optional `types` / `excludeTypes` filter which event kinds appear. Example: show every registered trigger except donations with `excludeTypes: ['donation']`.
- **folder** — absolute folder path; read-only display field, browse button, open-folder button (disabled until selected)
- **file** — absolute file path; read-only display, browse with `filters` / `filename` / `namePattern`, open in file manager (highlights the file when possible)
- **info** — read-only text block for hints and warnings; `editor.description` is the body; optional `editor.infoBorder`: `blue`, `red`, or `yellow`
- **spoiler** — no stored value; button that expands/collapses nested `items` inline (same schema entries as `GenerateConfig`, including rows and nested spoilers/pages)
- **page** — no stored value; button that opens a nested settings view for `items` with a Back control (recursion allowed)

Nested fields inside `spoiler` / `page` store values at the **parent params level** (sibling keys to the container), not under the container key.

Optional `visibleWhen: { key, equals }` hides a field in the settings UI unless the sibling params value matches (useful for custom-only fields).

## Spoiler / page example

```js
{
  key: 'advanced',
  type: 'spoiler',
  editor: {
    label: { en: 'Advanced', ru: 'Дополнительно', uk: 'Додатково' },
    description: { en: 'Optional tuning' },
  },
  items: [
    {
      key: 'debug',
      type: 'boolean',
      default: false,
      editor: { label: { en: 'Debug logging' } },
    },
    {
      key: 'network',
      type: 'page',
      editor: { label: { en: 'Network' } },
      items: [
        {
          key: 'timeout_ms',
          type: 'number',
          default: 5000,
          editor: { label: { en: 'Timeout (ms)' } },
        },
      ],
    },
  ],
},
```

## Info block example

```js
{
  key: 'mod_warning',
  type: 'info',
  editor: {
    label: { en: 'Important' },
    description: {
      en: 'This mod may trigger anti-cheat on some servers. Use at your own risk.',
    },
    infoBorder: 'yellow',
  },
},
```

## Path picker UI

`folder` and `file` fields render as a non-editable path field with **Browse** and **Open** actions. Open stays disabled until a path is chosen. For files, Open reveals the file in the system file manager.

## Example

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

## Triggers field example

Use a `triggers` field when the addon should react to dashboard events the user picks in settings (Twitch rewards, follows, and so on) — separately from any built-in donation handling.

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
      },
    },
  },
]);

events.On('onOrderTrigger', ({ key, trigger, record, user }) => {
  console.log(key, trigger.type, trigger.value, record, user);
});
```

`key` is the schema field key (`order_triggers`). `trigger` is the incoming dashboard event. Bindings are included in `triggers.getApplied()` as `addon` / `addonInput`.

## Reading params

```js
const params = await api.config.getParams();
await api.config.updateParams({ last_sync: Date.now() });
```

Size limit: **10 000** bytes JSON per addon unless `INCREASE_CONFIG_SIZE` is granted (**1 000 000** bytes).
