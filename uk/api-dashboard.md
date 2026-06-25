# dashboard

Живить віджет **останніх подій** і вікно **чату**. Потрібні `DASHBOARD_EVENTS` та/або `DASHBOARD_CHAT` (див. кожен метод).

## Реєстрація платформи

Викликайте під час завантаження, щоб UI міг розв'язувати id платформ:

```js
await dashboard.registerPlatform({
  id: 'myplatform',
  name: { en: 'My Platform', ru: 'Моя платформа', uk: 'Моя платформа' },
});
```

Використовуйте той самий `id` у полі `platform` для `addRecord` / `addChatMessage`.

## Користувачі

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

## Віджет подій — `addRecord`

**Потрібно:** `DASHBOARD_EVENTS`

```js
await dashboard.addRecord(
  {
    id: random.id(),
    type: 'donation', // donation | subscribe | follow | custom | timer
    platform: 'myplatform',
    amount: [10, 'USD'],
    message: { en: 'Thanks!', ru: 'Спасибо!' },
    from: 'user-1',
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform' },
  { trigger: { type: 'donation', key: 'USD', value: 10 } }, // optional overlay trigger
);
```

`message` приймає звичайний `string`, `{ en, ru?, uk? }` або кортеж app `LangData`.

Кілька тригерів: `{ triggers: [...] }`.

## Чат — `addChatMessage`

**Потрібно:** `DASHBOARD_CHAT`

```js
await dashboard.addChatMessage(
  {
    content: 'Hello chat!',
    platform: 'myplatform',
    from: 'user-1',
    emotes: [{ word: 'Kappa', url: 'https://example.com/kappa.png' }],
    style: {
      color: '#7c4dff',
      header: { en: 'Highlighted', uk: 'Виділено' },
      icon: 'megaphone',
    },
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform' },
);
```

`content` приймає звичайний `string` або `{ en, ru?, uk? }` — не кортежі `LangData` застосунку (аддони використовують власні об'єкти локалізації).

Опційний `style` додає кольорову рамку та опційну шапку:

| Поле | Опис |
| --- | --- |
| `color` | Колір рамки та фону шапки (CSS-колір, напр. `#ff9800`) |
| `header` | Текст шапки (`string` або `{ en, ru?, uk? }`); без поля — лише рамка |
| `icon` | `exclamation`, `question`, `megaphone` або `list` |

## Системний чат — `addSystemChatMessage`

**Потрібно:** `DASHBOARD_CHAT`

Системні рядки не від користувачів платформи. В UI показується іконка цього аддона. Опційний `sender` виводиться перед текстом.

```js
await dashboard.addSystemChatMessage({
  content: { en: 'Connected to chat', uk: 'Підключено до чату' },
  sender: { en: 'My addon' },
  style: {
    color: '#4caf50',
    header: { en: 'Notice' },
    icon: 'exclamation',
  },
});
```

`content`, `sender` і `style.header` приймають лише `string` або `{ en, ru?, uk? }`.

## Значки і емotes чату

```js
await dashboard.registerChatBadges([
  { id: 'badge-vip', url: 'https://example.com/vip.png', title: 'VIP' },
]);

await dashboard.registerChatEmotes({
  platforms: ['myplatform'],
  emotes: [{ word: 'hello', url: 'https://example.com/hello.png' }],
});
```

API читання (потрібні `DASHBOARD_CHAT` або `DASHBOARD_CHAT_INCOMING`):

- `listChatBadges()`
- `listChatEmotes()`
- `listPlatforms()`

## Відправка / вхідний чат

**Відправка (composer):** `DASHBOARD_CHAT`

```js
await dashboard.onChatSend(async ({ text }) => {
  // send text to your platform API
});
dashboard.offChatSend();
```

**Вхідні рядки:** `DASHBOARD_CHAT_INCOMING`

```js
dashboard.onChatMessage(msg => {
  console.log(msg.message.content, msg.user?.name, msg.sourceAddonId);
});
dashboard.offChatMessage();
```

## Вхідні події — `onRecord`

**Потрібно:** `DASHBOARD_EVENTS_INCOMING`

Підписка на нові записи у віджеті останніх подій. Payload містить збережений `record` (з системним `attach` для спрацьованих оверлеїв, звуків, хоткеїв і таймера), користувача, `triggers`, використані для матчингу, та `sourceAddonId`.

На відміну від чату, аддон-джерело також отримує свої записи — зручно перевіряти результат матчингу після `dashboard.addRecord`.

```js
dashboard.onRecord(payload => {
  console.log(payload.record.type, payload.record.attach, payload.triggers);
});
dashboard.offRecord();
```

## Тригери оверлею — `registerTriggers`

**Потрібно:** `DASHBOARD_EVENTS`

Реєструє типи подій, які користувач може прив'язати до оверлеїв у налаштуваннях. Передавайте відповідний `trigger` в опціях `addRecord`.

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

### Поля опцій тригера

| Поле | Опис |
| --- | --- |
| `type` | `donation`, `subscribe`, `subgift`, `follow`, `custom` |
| `key` | Фіксований дискримінатор (`bits`, `redeems`, …) |
| `label` | Локалізована назва в налаштуваннях оверлею |
| `valueType` | `text`, `number`, `select`, `dynamic` |
| `valueOptions` | Для `select` |
| `valueProvider` | Для `dynamic` — обробляйте події `overlayTriggerValue:{provider}:list\|create\|release` |
| `valueMatch` | `exact` (за замовчуванням) або `minimum` |
| `keyOptions` / `keyLabel` | Ключі, що обирає користувач (наприклад, валюта) |

### Події dynamic provider

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
    title: { uk: 'Нагороду створено' },
    message: { uk: `Вартість: ${context?.cost}` },
  },
}));
```

У відповідях можна передати `notify` — модальне вікно в налаштуваннях (`variant`: `success` | `error` | `info`; `title?`, `message`).

### Прив'язки тригерів після збереження налаштувань

Коли збережені правила для вашого аддона змінюються, main process викликає:

```js
events.On('triggers:applied-changed', ({ previous, current }) => {
  // previous / current групують правила за системами:
  // overlay, timer, game, gameInput, sounds, hotkeys
});
```

Використовуйте це, щоб звільняти внутрішні ресурси при видаленні прив'язок.

### Запит збережених прив'язок — `triggers.getApplied()`

Будь-який аддон може в будь-який момент запросити актуальну карту тригерів (без додаткових прав):

```js
const res = await triggers.getApplied();
if (res.success) {
  const { categories } = res;
  // categories.overlay, categories.timer, categories.game,
  // categories.gameInput, categories.sounds, categories.hotkeys
  // У кожній категорії: addonId → rules[]
  const twitchOverlay = categories.overlay['twitch'] || [];
  const allHotkeys = categories.hotkeys;
}
```

Містить прив'язки всіх аддонів (не лише того, що викликає). Ключі — id аддона-джерела подій дашборду, окрім `gameInput` (ключі — id ігрових аддонів).

Повний контракт — у JSDoc `registerTriggers` у згенерованих типах.
