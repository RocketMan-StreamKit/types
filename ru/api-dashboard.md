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
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform' },
  { trigger: { type: 'donation', key: 'USD', value: 10 } }, // опциональный триггер оверлея
);
```

`message` принимает простую `string`, `{ en, ru?, uk? }` или кортеж `LangData` приложения.

Несколько триггеров: `{ triggers: [...] }`.

## Чат — `addChatMessage`

**Требуется:** `DASHBOARD_CHAT`

```js
await dashboard.addChatMessage(
  {
    content: 'Hello chat!',
    platform: 'myplatform',
    from: 'user-1',
    color: '_as_user_',
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

API чтения (требуют `DASHBOARD_CHAT` или `DASHBOARD_CHAT_INCOMING`):

- `listChatBadges()`
- `listChatEmotes()`
- `listPlatforms()`

## Отправка / входящий чат

**Отправка (поле ввода):** `DASHBOARD_CHAT`

```js
await dashboard.onChatSend(async ({ text }) => {
  // отправка текста в API вашей платформы
});
dashboard.offChatSend();
```

**Входящие строки:** `DASHBOARD_CHAT_INCOMING`

```js
dashboard.onChatMessage(msg => {
  console.log(msg.message.content, msg.user?.name, msg.sourceAddonId);
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
