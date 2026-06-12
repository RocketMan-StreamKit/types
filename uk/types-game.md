# Ігрові інтеграції

`manifest.type` має бути `game`. Такі аддони запускають worker і пов’язують події стріму з ефектами в грі.

## Приклад manifest

```json
{
  "name": { "en": "GTA V integration", "uk": "Інтеграція GTA V" },
  "description": { "en": "In-game effects from viewers.", "uk": "Внутрішньоігрові ефекти від глядачів." },
  "type": "game",
  "version": "1.0.0",
  "author": "Your Name",
  "icon": "logo.png",
  "permissions": ["NETWORK_REQUEST"]
}
```

## Реєстрація внутрішньоігрових дій

Викличте один раз під час завантаження аддона:

```js
await game.registerInputTriggers([
  { id: 'spawn_car', label: { en: 'Spawn car', uk: 'Заспавнити авто' } },
  { id: 'explosion', label: { en: 'Explosion', uk: 'Вибух' } },
]);

events.On('gameInputTrigger', ({ actionId, trigger, record, user }) => {
  if (actionId === 'spawn_car') {
    // ефект у грі
  }
});
```

Користувач прив’язує події дашборду до дій у **Налаштування → Ігрові інтеграції**.

## Опційні тригери для оверлеїв

З правом `DASHBOARD_EVENTS` можна викликати `dashboard.registerTriggers()` і надсилати події через `dashboard.addRecord(..., { trigger })`.

## Поля шляху (`folder` / `file`)

```js
GenerateConfig([
  {
    key: 'game_exe',
    type: 'file',
    default: '',
    pathPicker: {
      title: { en: 'GTA V executable', uk: 'Виконуваний файл GTA V' },
      filename: 'GTAV.exe',
      filters: [{ name: 'Executable', extensions: ['exe'] }],
    },
    editor: { label: { en: 'Game executable', uk: 'Файл гри' }, required: true },
  },
]);
```

Див. також: [API dashboard](./api-dashboard.md), [Схема конфігурації](./config.md).
