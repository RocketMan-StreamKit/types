# dashboard

Питает виджет **последних событий** и окно **чата**. Требует `DASHBOARD_EVENTS` и/или `DASHBOARD_CHAT` (см. каждый метод).

## Регистрация платформы

Вызывайте при загрузке, чтобы UI мог разрешать id платформ:

```js
await dashboard.registerPlatform({
  id: 'myplatform',
  name: { en: 'My Platform', ru: 'Моя платформа', uk: 'Моя платформа' },
});
```

Используйте тот же `id` в поле `platform` для `addRecord` / `addChatMessage`.

## Пользователи

```js
await dashboard.upsertUser({
  id: 'user-1',
  name: 'Viewer',
  avatar: 'https://example.com/avatar.png',
  platform: 'myplatform',
  color: '#7fff00',
  icons: ['badge-vip'], // id из registerChatBadges
});
```

## Виджет событий — `addRecord`

**Требуется:** `DASHBOARD_EVENTS`

```js
await dashboard.addRecord(
  {
    id: random.id(),
    type: 'donation', // donation | subscribe | follow | custom | timer
    platform: 'myplatform',
    amount: [10, 'USD'],
    message: { en: 'Thanks!', ru: 'Спасибо!' },
    from: 'user-1',
    // Опциональные аттачи аддона (после registerAttaches)
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
  { trigger: { type: 'donation', key: 'USD', value: 10 } }, // опциональный триггер оверлея
);
```

`message` принимает простую `string`, `{ en, ru?, uk? }` или кортеж `LangData` приложения.

Несколько триггеров: `{ triggers: [...] }`.

### Типы аттачей — `registerAttaches`

**Требуется:** `DASHBOARD_EVENTS`

Регистрируйте типы аттачей при загрузке. `type` должен быть уникальным и не совпадать с системными (`overlay`, `sound`, `timer`, `hotkey`, `coop-sync`).

```js
await dashboard.registerAttaches([
  { type: 'clip', label: { en: 'Clip', ru: 'Клип', uk: 'Кліп' } },
]);
```

Передавайте `attach` в `addRecord` или обновляйте существующую запись:

```js
await dashboard.updateRecordAttaches('record-id', [
  {
    type: 'clip',
    value: { en: 'Funny moment', ru: 'Забавный момент' },
    id: 'clip-1',
    playable: true,
    playing: true,
  },
], { mode: 'merge' }); // или 'replace', чтобы сначала сбросить аттачи этого аддона
```

| Поле | Описание |
| --- | --- |
| `type` | Зарегистрированный тип аттача |
| `value` | Отображаемый текст (`string` или `{ en, ru?, uk? }`) |
| `id` | Идентификатор для кнопки проигрывания (не обязан быть уникальным); иначе берётся строковый `value` / `value.en` |
| `playable` | `true` — показать кнопку play/stop |
| `playing` | `true` — кнопка в состоянии stop |

### Проигрывание аттача — `onAttachPlay`

**Требуется:** `DASHBOARD_EVENTS`

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

В payload: `id`, `type`, `action`, полная запись `record`, а также `recordId` / timestamps.

## Чат — `addChatMessage`

**Требуется:** `DASHBOARD_CHAT`

```js
await dashboard.addChatMessage(
  {
    content: 'Hello chat!',
    platform: 'myplatform',
    from: 'user-1',
    color: '_as_user_',
    // Опционально: пометить как системное для других аддонов (без смены UI)
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

`content` принимает простую `string` или `{ en, ru?, uk? }` — не кортежи `LangData` приложения (аддоны используют свои объекты локализации).

Опциональный `system: true` помечает сообщение как системное для аддонов. Другие аддоны получают его как `msg.message.system` в `onChatMessage`. Окно чата по-прежнему показывает обычную пользовательскую строку.

Опциональный `color` задаёт цвет **текста сообщения** (не рамки). Допустимые значения:

| Значение | Пример | Результат |
| --- | --- | --- |
| Hex с `#` | `'#ff0000'`, `'#f00'` | Нормализуется в `#rrggbb` |
| Hex без `#` | `'ff0000'`, `'f00'` | То же |
| RGB-кортеж | `[255, 128, 0]` | `#ff8000` |
| RGB-объект | `{ r: 0, g: 0, b: 255 }` | `#0000ff` |
| `_as_user_` | `'_as_user_'` | Цвет ника автора (`user.color`) |

Некорректные значения игнорируются. Каналы вне `0…255` ограничиваются.

Опциональный `style` добавляет цветную рамку и опциональную шапку:

| Поле | Описание |
| --- | --- |
| `color` | Цвет рамки и фона шапки (CSS-цвет, например `#ff9800`) |
| `header` | Текст шапки (`string` или `{ en, ru?, uk? }`); без поля — только рамка |
| `icon` | `exclamation`, `question`, `megaphone` или `list` |

## Системный чат — `addSystemChatMessage`

**Требуется:** `DASHBOARD_CHAT`

Системные строки не от пользователей платформы. В UI показывается иконка этого аддона. Опциональный `sender` выводится перед текстом.

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

`content`, `sender` и `style.header` принимают только `string` или `{ en, ru?, uk? }`. Опциональный `color` — те же форматы, что у `addChatMessage` (hex / RGB / `_as_user_`).

## Значки и эмоуты чата

```js
await dashboard.registerChatBadges([
  { id: 'badge-vip', url: 'https://example.com/vip.png', title: 'VIP' },
]);

await dashboard.registerChatEmotes({
  platforms: ['myplatform'],
  emotes: [{ word: 'hello', url: 'https://example.com/hello.png' }],
});
```

При сохранении сообщения StreamKit+ ищет в тексте зарегистрированные эмоуты платформы и дописывает совпадения в `message.emotes` (записи от отправителя имеют приоритет при конфликте). Они остаются в сохранённом сообщении и в payload `onChatMessage` даже после удаления аддона, который их зарегистрировал.

API чтения (требуют `DASHBOARD_CHAT` или `DASHBOARD_CHAT_INCOMING`):

- `listChatBadges()`
- `listChatEmotes()`
- `listPlatforms()`

## Отправка / входящий чат

**Приём из поля ввода (стать целью отправки):** `DASHBOARD_CHAT`

```js
await dashboard.onChatSend(async ({ text, system }) => {
  // `system` — назначение сообщения (не оформление в UI):
  // - не указан / false — от имени стримера (окно чата никогда не ставит system)
  // - true — системное/автоматическое (например ответ бота через аккаунт бота)
  if (system) {
    // отправить от бота / автоматического аккаунта
  } else {
    // отправить от стримера
  }
});
dashboard.offChatSend();
```

**Исходящая отправка (тот же путь, что у окна чата):** `DASHBOARD_CHAT_SEND`

Второй аргумент можно не указывать — сообщение уйдёт через **все** зарегистрированные подписчики `onChatSend`. Массив строк — только выбранные id аддонов.

Третий аргумент `{ system?: boolean }` задаёт назначение сообщения для обработчиков `onChatSend`. По умолчанию `false` (от стримера). `system: true` — для автоматических/бот-сообщений. Поле ввода окна чата никогда не передаёт `system`.

```js
// Через все аддоны, подписанные на onChatSend
await dashboard.sendChatMessage('Hello everyone!');

// Через конкретные аддоны
await dashboard.sendChatMessage('Hi Twitch!', ['twitch']);
await dashboard.sendChatMessage('Multi', ['twitch', 'kick']);

// Системная / автоматическая отправка (например аккаунт бота)
await dashboard.sendChatMessage('Thanks for the follow!', ['twitch'], {
  system: true,
});
```

Возвращает `{ success: true }`, если хотя бы одна цель приняла сообщение, или `{ success: false, message }`, если текст пустой, нет валидных целей или все цели завершились ошибкой.

**Входящие строки:** `DASHBOARD_CHAT_INCOMING`

```js
dashboard.onChatMessage(msg => {
  console.log(msg.message.content, msg.message.system, msg.user?.name, msg.sourceAddonId);
});
dashboard.offChatMessage();
```

## Входящие события — `onRecord`

**Требуется:** `DASHBOARD_EVENTS_INCOMING`

Подписка на новые записи в виджете последних событий. Payload содержит сохранённый `record` (с системным `attach` для сработавших оверлеев, звуков, хоткеев и таймера), пользователя, `triggers`, использованные для матчинга, и `sourceAddonId`.

В отличие от чата, аддон-источник тоже получает свои записи — удобно проверять результат матчинга после `dashboard.addRecord`.

```js
dashboard.onRecord(payload => {
  console.log(payload.record.type, payload.record.attach, payload.triggers);
});
dashboard.offRecord();
```

## Триггеры оверлея — `registerTriggers`

**Требуется:** `DASHBOARD_EVENTS`

Регистрирует типы событий, которые пользователь может привязать к оверлеям в настройках. Передавайте соответствующий `trigger` в опциях `addRecord`.

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

### Поля опции триггера

| Поле | Описание |
| --- | --- |
| `type` | `donation`, `subscribe`, `subgift`, `follow`, `custom` |
| `key` | Фиксированный дискриминатор (`bits`, `redeems`, …) |
| `label` | Локализованное имя в настройках оверлея |
| `valueType` | `text`, `number`, `select`, `dynamic` |
| `valueOptions` | Для `select` |
| `valueProvider` | Для `dynamic` — обработка событий `overlayTriggerValue:{provider}:list\|create\|release` |
| `valueMatch` | `exact` (по умолчанию) или `minimum` |
| `keyOptions` / `keyLabel` | Выбираемые пользователем ключи (например, валюта) |

### События dynamic provider

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
    title: { ru: 'Награда создана' },
    message: { ru: `Стоимость: ${context?.cost}` },
  },
}));
```

В ответах можно передать `notify` — модальное окно в настройках (`variant`: `success` | `error` | `info`; `title?`, `message`).

### Привязки триггеров после сохранения настроек

Когда сохранённые правила для вашего аддона меняются, main process вызывает:

```js
events.On('triggers:applied-changed', ({ previous, current }) => {
  // previous / current группируют правила по системам:
  // overlay, timer, game, gameInput, sounds, hotkeys
});
```

Используйте это, чтобы освобождать внутренние ресурсы при удалении привязок.

### Валидация привязок перед сохранением настроек

Перед записью настроек, связанных с триггерами, приложение вызывает каждый связанный аддон:

```js
events.On('triggers:validate', ({ draft }) => {
  for (const rule of draft.overlay || []) {
    if (rule.trigger.key === 'redeems' && !String(rule.trigger.value || '').trim()) {
      return {
        success: false,
        message: 'Сначала сгенерируйте или выберите награду за баллы канала',
      };
    }
  }
  return { success: true };
});
```

Верните `{ success: false, message }`, чтобы заблокировать сохранение. Если обработчика нет (или он вернул success), сохранение продолжается. Дополнительных прав не требуется.

### Запрос сохранённых привязок — `triggers.getApplied()`

Любой аддон может в любой момент запросить актуальную карту триггеров (без дополнительных прав):

```js
const res = await triggers.getApplied();
if (res.success) {
  const { categories } = res;
  // categories.overlay, categories.timer, categories.game,
  // categories.gameInput, categories.sounds, categories.hotkeys
  // В каждой категории: addonId → rules[]
  const twitchOverlay = categories.overlay['twitch'] || [];
  const allHotkeys = categories.hotkeys;
}
```

Включает привязки всех аддонов (не только вызывающего). Ключи — id аддона-источника событий дашборда, кроме `gameInput` (ключи — id игровых аддонов).

### Удаление своих привязок — `triggers.removeApplied()`

Удаляет сохранённые правила, где этот аддон — источник событий дашборда (и связанные overlay/game target-правила), без дополнительных прав:

```js
await triggers.removeApplied(); // все системы
await triggers.removeApplied({ systems: ['sounds', 'hotkeys'] });
```

После этого освобождаются неиспользуемые managed dynamic values.

Полный контракт — в JSDoc `registerTriggers` в сгенерированных типах.
