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
    emotes: [{ word: 'Kappa', url: 'https://example.com/kappa.png' }],
  },
  { id: 'user-1', name: 'Viewer', platform: 'myplatform' },
);
```

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
```

Полный контракт — в JSDoc `registerTriggers` в сгенерированных типах.
