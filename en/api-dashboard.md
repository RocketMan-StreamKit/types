# dashboard

Feeds the **latest-events** widget and **chat** window. Requires `DASHBOARD_EVENTS` and/or `DASHBOARD_CHAT` (see each method).

## Platform registration

Call at load time so the UI can resolve platform ids:

```js
await dashboard.registerPlatform({
  id: 'myplatform',
  name: { en: 'My Platform', ru: 'Моя платформа', uk: 'Моя платформа' },
});
```

Use the same `id` in `addRecord` / `addChatMessage` `platform` field.

## Users

```js
await dashboard.upsertUser({
  id: 'user-1',
  name: 'Viewer',
  avatar: 'https://example.com/avatar.png',
  platform: 'myplatform',
  color: '#7fff00',
  icons: ['badge-vip'], // ids from registerChatBadges
});
```

## Events widget — `addRecord`

**Requires:** `DASHBOARD_EVENTS`

```js
await dashboard.addRecord(
  {
    id: random.id(),
    type: 'donation', // donation | subscribe | follow | custom | timer
    platform: 'myplatform',
    amount: [10, 'USD'],
    message: { en: 'Thanks!', ru: 'Спасибо!' },
    from: 'user-1',
    // Optional addon attaches (after registerAttaches)
    attach: [
      {
        type: 'clip',
        value: { en: 'Funny moment', ru: 'Забавный момент', uk: 'Кумедний момент' },
        id: 'clip-1',
        playable: true,
        playing: false,
      },
    ],
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform' },
  { trigger: { type: 'donation', key: 'USD', value: 10 } }, // optional overlay trigger
);
```

`message` accepts plain `string`, `{ en, ru?, uk? }`, or app `LangData` tuple.

Multiple triggers: `{ triggers: [...] }`.

### Attach types — `registerAttaches`

**Requires:** `DASHBOARD_EVENTS`

Register attach types at load time. `type` must be unique and must not use system ids (`overlay`, `sound`, `timer`, `hotkey`, `coop-sync`).

```js
await dashboard.registerAttaches([
  { type: 'clip', label: { en: 'Clip', ru: 'Клип', uk: 'Кліп' } },
]);
```

Pass matching `attach` entries in `addRecord`, or update an existing row:

```js
await dashboard.updateRecordAttaches('record-id', [
  {
    type: 'clip',
    value: { en: 'Funny moment', ru: 'Забавный момент' },
    id: 'clip-1',
    playable: true,
    playing: true,
  },
], { mode: 'merge' }); // or 'replace' to drop this addon's attaches first
```

| Field | Description |
| --- | --- |
| `type` | Registered attach type |
| `value` | Display text (`string` or `{ en, ru?, uk? }`) |
| `id` | Play-button identifier (not required to be unique); falls back to string `value` / `value.en` |
| `playable` | When `true`, shows a play/stop button |
| `playing` | When `true`, the button shows the stop state |

### Attach play — `onAttachPlay`

**Requires:** `DASHBOARD_EVENTS`

```js
dashboard.onAttachPlay(async ({ id, type, action, record }) => {
  // action: 'play' | 'stop'
  await dashboard.updateRecordAttaches(record.id, [
    {
      type,
      value: { en: 'Funny moment', ru: 'Забавный момент' },
      id,
      playable: true,
      playing: action === 'play',
    },
  ]);
});
dashboard.offAttachPlay();
```

The payload includes attach `id`, `type`, `action`, the full stored `record`, plus `recordId` / timestamps.

## Chat — `addChatMessage`

**Requires:** `DASHBOARD_CHAT`

```js
await dashboard.addChatMessage(
  {
    content: 'Hello chat!',
    platform: 'myplatform',
    from: 'user-1',
    color: '_as_user_',
    // Optional: mark as system for other addons (no UI change)
    system: true,
    emotes: [{ word: 'Kappa', url: 'https://example.com/kappa.png' }],
    style: {
      color: '#7c4dff',
      header: { en: 'Highlighted', ru: 'Выделено' },
      icon: 'megaphone',
    },
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform', color: '#9147ff' },
);
```

`content` accepts plain `string` or `{ en, ru?, uk? }` — not app `LangData` tuples (addons use their own localized objects).

Optional `system: true` marks the message as system for addons. Other addons receive it as `msg.message.system` in `onChatMessage`. The chat window still renders a normal user line.

Optional `color` sets the **message text** color (not the border). Accepted values:

| Value | Example | Result |
| --- | --- | --- |
| Hex with `#` | `'#ff0000'`, `'#f00'` | Normalized to `#rrggbb` |
| Hex without `#` | `'ff0000'`, `'f00'` | Same |
| RGB tuple | `[255, 128, 0]` | `#ff8000` |
| RGB object | `{ r: 0, g: 0, b: 255 }` | `#0000ff` |
| `_as_user_` | `'_as_user_'` | Same color as the author's nickname (`user.color`) |

Invalid values are ignored. Channels outside `0…255` are clamped.

Optional `style` adds a colored border and optional header bar:

| Field | Description |
| --- | --- |
| `color` | Border and header background (CSS color, e.g. `#ff9800`) |
| `header` | Header text (`string` or `{ en, ru?, uk? }`); omit for border only |
| `icon` | `exclamation`, `question`, `megaphone`, or `list` |

## System chat — `addSystemChatMessage`

**Requires:** `DASHBOARD_CHAT`

System lines are not from platform users. The UI shows this addon's icon. Optional `sender` is shown before the message text.

```js
await dashboard.addSystemChatMessage({
  content: { en: 'Connected to chat', ru: 'Подключено к чату' },
  sender: { en: 'My addon' },
  color: '#4caf50',
  style: {
    color: '#4caf50',
    header: { en: 'Notice' },
    icon: 'exclamation',
  },
});
```

`content`, `sender`, and `style.header` accept `string` or `{ en, ru?, uk? }` only. Optional `color` uses the same formats as `addChatMessage` (hex / RGB / `_as_user_`).

## Chat badges and emotes

```js
await dashboard.registerChatBadges([
  { id: 'badge-vip', url: 'https://example.com/vip.png', title: 'VIP' },
]);

await dashboard.registerChatEmotes({
  platforms: ['myplatform'],
  emotes: [{ word: 'hello', url: 'https://example.com/hello.png' }],
});
```

When a chat message is saved, StreamKit+ scans its text for registered emotes on that platform and merges matches into `message.emotes` (entries already passed by the sender win on collisions). Those emotes stay on the stored message and in `onChatMessage` payloads even after the registering addon is removed.

Read APIs (require `DASHBOARD_CHAT` or `DASHBOARD_CHAT_INCOMING`):

- `listChatBadges()`
- `listChatEmotes()`
- `listPlatforms()`

## Chat send / incoming

**Receive composer sends (be a send target):** `DASHBOARD_CHAT`

```js
await dashboard.onChatSend(async ({ text }) => {
  // send text to your platform API
});
dashboard.offChatSend();
```

**Dispatch outgoing text (same path as the chat window):** `DASHBOARD_CHAT_SEND`

Omit the second argument to send through **all** registered chat-send subscribers. Pass a string array to target specific addon ids.

```js
// Through every addon that called onChatSend
await dashboard.sendChatMessage('Hello everyone!');

// Through specific addons only
await dashboard.sendChatMessage('Hi Twitch!', ['twitch']);
await dashboard.sendChatMessage('Multi', ['twitch', 'kick']);
```

Returns `{ success: true }` when at least one target accepts the message, or `{ success: false, message }` when the text is empty, no valid targets are registered, or every target fails.

**Incoming lines:** `DASHBOARD_CHAT_INCOMING`

```js
dashboard.onChatMessage(msg => {
  console.log(msg.message.content, msg.message.system, msg.user?.name, msg.sourceAddonId);
});
dashboard.offChatMessage();
```

## Events incoming — `onRecord`

**Requires:** `DASHBOARD_EVENTS_INCOMING`

Subscribe to new records in the latest-events dashboard widget. The payload includes the stored `record` (with system `attach` entries for matched overlays, sounds, hotkeys, and timers), resolved user, `triggers` used for matching, and `sourceAddonId`.

Unlike chat incoming, the source addon also receives its own records — useful to inspect match results after `dashboard.addRecord`.

```js
dashboard.onRecord(payload => {
  console.log(payload.record.type, payload.record.attach, payload.triggers);
});
dashboard.offRecord();
```

## Overlay triggers — `registerTriggers`

**Requires:** `DASHBOARD_EVENTS`

Registers event types users can bind to overlays in settings. Pass matching `trigger` in `addRecord` options.

```js
await dashboard.registerTriggers([
  {
    type: 'follow',
    label: { en: 'New follower', ru: 'Новый фолловер' },
  },
  {
    type: 'custom',
    key: 'bits',
    label: { en: 'Cheer (bits)' },
    valueType: 'number',
    valueMatch: 'minimum',
    valueHint: { en: 'Minimum bits' },
  },
]);
```

### Trigger option fields

| Field | Description |
| --- | --- |
| `type` | `donation`, `subscribe`, `subgift`, `follow`, `custom` |
| `key` | Fixed discriminator (`bits`, `redeems`, …) |
| `label` | Localized name in overlay settings |
| `valueType` | `text`, `number`, `select`, `dynamic` |
| `valueOptions` | For `select` |
| `valueProvider` | For `dynamic` — handle `overlayTriggerValue:{provider}:list\|create\|release` events |
| `valueMatch` | `exact` (default) or `minimum` |
| `keyOptions` / `keyLabel` | User-selectable keys (e.g. currency) |

### Dynamic provider events

```js
events.On('overlayTriggerValue:rewards:list', async () => ({
  success: true,
  items: [{ id: 'abc', label: 'My reward', meta: '100' }],
}));

events.On('overlayTriggerValue:rewards:create', async ({ title, context }) => ({
  success: true,
  valueId: 'abc',
  label: title,
  notify: {
    variant: 'success',
    title: { en: 'Reward created' },
    message: { en: `Cost: ${context?.cost}` },
  },
}));
```

Responses may include optional `notify` — a modal in settings (`variant`: `success` | `error` | `info`; `title?`, `message`).

### Trigger bindings after settings save

When saved trigger rules for your addon change, the main process fires:

```js
events.On('triggers:applied-changed', ({ previous, current }) => {
  // previous / current group rules by system:
  // overlay, timer, game, gameInput, sounds, hotkeys
});
```

Use this to release internal resources when bindings are removed.

### Validate trigger bindings before settings save

Before the app persists trigger-related settings, it calls each related addon:

```js
events.On('triggers:validate', ({ draft }) => {
  for (const rule of draft.overlay || []) {
    if (rule.trigger.key === 'redeems' && !String(rule.trigger.value || '').trim()) {
      return {
        success: false,
        message: 'Generate or select a channel point reward first',
      };
    }
  }
  return { success: true };
});
```

Return `{ success: false, message }` to block the save. Omitting the handler (or returning success) allows the save. No extra permission is required.

### Query saved trigger bindings — `triggers.getApplied()`

Any addon can request the current trigger map at any time (no extra permission):

```js
const res = await triggers.getApplied();
if (res.success) {
  const { categories } = res;
  // categories.overlay, categories.timer, categories.game,
  // categories.gameInput, categories.sounds, categories.hotkeys
  // Each is a map: addonId → rules[]
  const twitchOverlay = categories.overlay['twitch'] || [];
  const allHotkeys = categories.hotkeys;
}
```

Includes bindings for every addon (not only the caller). Keys in each category are dashboard event source addon ids, except `gameInput` where keys are game addon ids.

### Delete this addon's trigger bindings — `triggers.removeApplied()`

Remove persisted rules where this addon is the dashboard source (and related overlay/game target bindings) without extra permissions:

```js
await triggers.removeApplied(); // every system
await triggers.removeApplied({ systems: ['sounds', 'hotkeys'] });
```

Unreferenced managed dynamic values are released afterward.

See JSDoc on `registerTriggers` in generated typings for full contract.
