# Settings schema (`GenerateConfig`)

Call `GenerateConfig([...])` at addon load time to declare settings fields. Schemas are stored in app config; values live in `addonsParams[id]`. Defaults are merged on registration and when the settings page opens.

## Field structure

Each field has:

| Property | Description |
| --- | --- |
| `key` | Storage key in params object |
| `type` | `text`, `hidden`, `color`, `number`, `boolean`, `array`, `object`, `select`, `button`, `folder`, `file`, `info` |
| `pathPicker` | For `folder` / `file`: `title`, `filters` (file only), `filename` (exact basename), `namePattern` (regex on basename, file only) |
| `default` | Initial value |
| `editor` | If present, field appears in settings UI with label, validation, etc. |
| `options` | For `select`: `{ value, label? }[]` |
| `items` | For `array`: `'text'` or `'number'` |
| `fields` | For `object`: nested fields |
| `event` | For `button`: event name fired on click (handle with `events.On`) |

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

- **text** — string
- **hidden** — string (same storage and validation as `text`); masked in the UI with a Show/Hide toggle (password field by default)
- **color** — hex string (`#ff1744`)
- **number** — number
- **boolean** — boolean
- **array** — array of text or numbers (`items: 'text' | 'number'`)
- **object** — nested object with `fields`
- **select** — `options: { value, label? }[]`; `default` must match an option or the first option is used
- **button** — no stored value; triggers `events.On(event, …)` when clicked (gated to declared event names)
- **folder** — absolute folder path; read-only display field, browse button, open-folder button (disabled until selected)
- **file** — absolute file path; read-only display, browse with `filters` / `filename` / `namePattern`, open in file manager (highlights the file when possible)
- **info** — read-only text block for hints and warnings; `editor.description` is the body; optional `editor.infoBorder`: `blue`, `red`, or `yellow`

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

## Reading params

```js
const params = await api.config.getParams();
await api.config.updateParams({ last_sync: Date.now() });
```

Size limit: **10 000** bytes JSON per addon unless `INCREASE_CONFIG_SIZE` is granted (**1 000 000** bytes).
