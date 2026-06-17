# Game integration addons

`manifest.type` must be `game`. These addons run a worker and connect stream events to in-game effects.

## Manifest example

```json
{
  "name": { "en": "GTA V integration" },
  "description": { "en": "Viewer-triggered in-game effects." },
  "type": "game",
  "version": "1.0.0",
  "author": "Your Name",
  "icon": "logo.png",
  "permissions": ["NETWORK_REQUEST"]
}
```

## Register in-game actions

Call once at addon load:

```js
await game.registerInputTriggers([
  { id: 'spawn_car', label: { en: 'Spawn car', ru: 'Заспавнить машину' } },
  { id: 'explosion', label: { en: 'Explosion', ru: 'Взрыв' } },
]);

events.On('gameInputTrigger', ({ actionId, trigger, record, user }) => {
  if (actionId === 'spawn_car') {
    // run in-game effect
  }
});
```

Users bind dashboard events to each action in **Settings → Game integrations**.

## Optional overlay triggers

With `DASHBOARD_EVENTS`, you can also call `dashboard.registerTriggers()` and push events via `dashboard.addRecord(..., { trigger })` so overlays and other modules react to game events.

## Path settings (`folder` / `file`)

```js
GenerateConfig([
  {
    key: 'game_exe',
    type: 'file',
    default: '',
    pathPicker: {
      title: { en: 'GTA V executable' },
      filename: 'GTAV.exe',
      filters: [{ name: 'Executable', extensions: ['exe'] }],
    },
    editor: { label: { en: 'Game executable' }, required: true },
  },
]);
```

See also: [API dashboard](./api-dashboard.md), [Config schema](./config.md).
