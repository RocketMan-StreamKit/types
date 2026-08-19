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
    // Опційні аттачі аддона (після registerAttaches)
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

`message` приймає звичайний `string`, `{ en, ru?, uk? }` або кортеж app `LangData`.

Кілька тригерів: `{ triggers: [...] }`.

### Типи аттачів — `registerAttaches`

**Потрібно:** `DASHBOARD_EVENTS`

Реєструйте типи аттачів під час завантаження. `type` має бути унікальним і не збігатися із системними (`overlay`, `sound`, `timer`, `hotkey`, `coop-sync`).

```js
await dashboard.registerAttaches([
  { type: 'clip', label: { en: 'Clip', ru: 'Клип', uk: 'Кліп' } },
]);
```

Передавайте `attach` у `addRecord` або оновлюйте наявний запис:

```js
await dashboard.updateRecordAttaches('record-id', [
  {
    type: 'clip',
    value: { en: 'Funny moment', uk: 'Кумедний момент' },
    id: 'clip-1',
    playable: true,
    playing: true,
  },
], { mode: 'merge' }); // або 'replace', щоб спочатку скинути аттачі цього аддона
```

| Поле | Опис |
| --- | --- |
| `type` | Зареєстрований тип аттача |
| `value` | Текст для відображення (`string` або `{ en, ru?, uk? }`) |
| `id` | Ідентифікатор для кнопки відтворення (не обов’язково унікальний); інакше береться рядковий `value` / `value.en` |
| `playable` | `true` — показати кнопку play/stop |
| `playing` | `true` — кнопка у стані stop |

### Відтворення аттача — `onAttachPlay`

**Потрібно:** `DASHBOARD_EVENTS`

```js
dashboard.onAttachPlay(async ({ id, type, action, record }) => {
  // action: 'play' | 'stop'
  await dashboard.updateRecordAttaches(record.id, [
    {
      type,
      value: { en: 'Funny moment', uk: 'Кумедний момент' },
      id,
      playable: true,
      playing: action === 'play',
    },
  ]);
});
dashboard.offAttachPlay();
```

У payload: `id`, `type`, `action`, повний запис `record`, а також `recordId` / timestamps.

## Чат — `addChatMessage`

**Потрібно:** `DASHBOARD_CHAT`

```js
await dashboard.addChatMessage(
  {
    content: 'Hello chat!',
    platform: 'myplatform',
    from: 'user-1',
    color: '_as_user_',
    // Опційно: позначити як системне для інших аддонів (індикатор у чаті)
    system: true,
    // Опційно: виділити як згадку стрімера у вікні чату
    mention: true,
    emotes: [{ word: 'Kappa', url: 'https://example.com/kappa.png' }],
    style: {
      color: '#7c4dff',
      header: { en: 'Highlighted', uk: 'Виділено' },
      icon: 'megaphone',
    },
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform', color: '#9147ff' },
);
```

`content` приймає звичайний `string` або `{ en, ru?, uk? }` — не кортежі `LangData` застосунку (аддони використовують власні об'єкти локалізації).

Опційний `system: true` позначає повідомлення як системне для аддонів. Інші аддони отримують його як `msg.message.system` у `onChatMessage`. Вікно чату й надалі показує звичайний рядок користувача та індикатор системного повідомлення після іконки платформи.

Опційний `mention: true` позначає повідомлення як згадку стрімера. Інші аддони отримують його як `msg.message.mention` у `onChatMessage`. Вікно чату виділяє рядок бурштиновим акцентом.

Опційний `color` задає колір **тексту повідомлення** (не рамки). Допустимі значення:

| Значення | Приклад | Результат |
| --- | --- | --- |
| Hex з `#` | `'#ff0000'`, `'#f00'` | Нормалізується до `#rrggbb` |
| Hex без `#` | `'ff0000'`, `'f00'` | Те саме |
| RGB-кортеж | `[255, 128, 0]` | `#ff8000` |
| RGB-об'єкт | `{ r: 0, g: 0, b: 255 }` | `#0000ff` |
| `_as_user_` | `'_as_user_'` | Колір ніка автора (`user.color`) |

Некоректні значення ігноруються. Канали поза `0…255` обмежуються.

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
  color: '#4caf50',
  style: {
    color: '#4caf50',
    header: { en: 'Notice' },
    icon: 'exclamation',
  },
});
```

`content`, `sender` і `style.header` приймають лише `string` або `{ en, ru?, uk? }`. Опційний `color` — ті самі формати, що й у `addChatMessage` (hex / RGB / `_as_user_`).

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

Під час збереження повідомлення StreamKit+ шукає в тексті зареєстровані емоути платформи й дописує збіги в `message.emotes` (записи від відправника мають пріоритет при конфлікті). Вони лишаються у збереженому повідомленні та в payload `onChatMessage` навіть після видалення аддона, який їх зареєстрував.

API читання (потрібні `DASHBOARD_CHAT` або `DASHBOARD_CHAT_INCOMING`):

- `listChatBadges()`
- `listChatEmotes()`
- `listPlatforms()`

## Відправка / вхідний чат

**Прийом з поля вводу (стати ціллю відправки):** `DASHBOARD_CHAT`

```js
await dashboard.onChatSend(async ({ text, system }) => {
  // `system` — призначення повідомлення (не оформлення в UI):
  // - не вказано / false — від імені стрімера (вікно чату ніколи не ставить system)
  // - true — системне/автоматичне (наприклад відповідь бота через акаунт бота)
  if (system) {
    // надіслати від бота / автоматичного акаунта
  } else {
    // надіслати від стрімера
  }
});
dashboard.offChatSend();
```

**Вихідна відправка (той самий шлях, що й у вікна чату):** `DASHBOARD_CHAT_SEND`

Другий аргумент можна не вказувати — повідомлення піде через **усі** зареєстровані підписники `onChatSend`. Масив рядків — лише вибрані id аддонів.

Третій аргумент `{ system?: boolean }` задає призначення повідомлення для обробників `onChatSend`. За замовчуванням `false` (від стрімера). `system: true` — для автоматичних/бот-повідомлень. Поле вводу вікна чату ніколи не передає `system`.

```js
// Через усі аддони, підписані на onChatSend
await dashboard.sendChatMessage('Hello everyone!');

// Через конкретні аддони
await dashboard.sendChatMessage('Hi Twitch!', ['twitch']);
await dashboard.sendChatMessage('Multi', ['twitch', 'kick']);

// Системна / автоматична відправка (наприклад акаунт бота)
await dashboard.sendChatMessage('Thanks for the follow!', ['twitch'], {
  system: true,
});
```

Повертає `{ success: true }`, якщо хоча б одна ціль прийняла повідомлення, або `{ success: false, message }`, якщо текст порожній, немає валідних цілей або всі цілі завершились помилкою.

**Вхідні рядки:** `DASHBOARD_CHAT_INCOMING`

```js
dashboard.onChatMessage(msg => {
  console.log(msg.message.content, msg.message.system, msg.message.mention, msg.user?.name, msg.sourceAddonId);
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

Використовуйте це, щоб звільняти внутрішні ресурси при видаленні прив'язок **або** коли споживач стає неактивним. Правила overlay і game мають `enabled` (`false`, якщо оверлей/ігрову інтеграцію вимкнено, зупинено після збоїв або обидва display-канали оверлею вимкнено). У звуків і хоткеїв `enabled` уже береться з їхніх перемикачів. Подія також надходить при зміні прапорців показу оверлею або вмиканні/вимиканні аддона, а не лише при редагуванні рядків тригерів.

### Валідація прив'язок перед збереженням налаштувань

Перед записом налаштувань, пов'язаних із тригерами, застосунок викликає кожен пов'язаний аддон:

```js
events.On('triggers:validate', ({ draft }) => {
  for (const rule of draft.overlay || []) {
    if (rule.trigger.key === 'redeems' && !String(rule.trigger.value || '').trim()) {
      return {
        success: false,
        message: 'Спочатку згенеруйте або виберіть нагороду за бали каналу',
      };
    }
  }
  return { success: true };
});
```

Поверніть `{ success: false, message }`, щоб заблокувати збереження. Якщо обробника немає (або він повернув success), збереження триває. Додаткових прав не потрібно.

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

### Видалення своїх прив'язок — `triggers.removeApplied()`

Видаляє збережені правила, де цей аддон — джерело подій дашборду (і пов'язані overlay/game target-правила), без додаткових прав:

```js
await triggers.removeApplied(); // усі системи
await triggers.removeApplied({ systems: ['sounds', 'hotkeys'] });
```

Після цього звільняються невикористані managed dynamic values.

Повний контракт — у JSDoc `registerTriggers` у згенерованих типах.
