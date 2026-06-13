# Игровые интеграции

`manifest.type` должен быть `game`. Такие аддоны запускают worker и связывают события стрима с эффектами в игре.

## Пример manifest

```json
{
  "name": { "en": "GTA V integration", "ru": "Интеграция GTA V" },
  "description": { "en": "In-game effects from viewers.", "ru": "Внутриигровые эффекты от зрителей." },
  "type": "game",
  "version": "1.0.0",
  "author": "Your Name",
  "icon": "logo.png",
  "permissions": ["NETWORK_REQUEST"]
}
```

## Регистрация внутриигровых действий

Вызовите один раз при загрузке аддона:

```js
await game.registerInputTriggers([
  { id: 'spawn_car', label: { en: 'Spawn car', ru: 'Заспавнить машину' } },
  { id: 'explosion', label: { en: 'Explosion', ru: 'Взрыв' } },
]);

events.On('gameInputTrigger', ({ actionId, trigger, record, user }) => {
  if (actionId === 'spawn_car') {
    // эффект в игре
  }
});
```

Пользователь привязывает события дашборда к действиям в **Настройки → Игровые интеграции**.

## Опциональные триггеры для оверлеев

С правом `DASHBOARD_EVENTS` можно вызывать `dashboard.registerTriggers()` и отправлять события через `dashboard.addRecord(..., { trigger })`.

## Поля пути (`folder` / `file`)

```js
GenerateConfig([
  {
    key: 'game_exe',
    type: 'file',
    default: '',
    pathPicker: {
      title: { en: 'GTA V executable', ru: 'Исполняемый файл GTA V' },
      filename: 'GTAV.exe',
      filters: [{ name: 'Executable', extensions: ['exe'] }],
    },
    editor: { label: { en: 'Game executable', ru: 'Файл игры' }, required: true },
  },
]);
```

См. также: [API dashboard](./api-dashboard.md), [Схема конфигурации](./config.md).
