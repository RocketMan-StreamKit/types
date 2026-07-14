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
  "permissions": ["NETWORK_REQUEST", "FILE_ACCESS"]
}
```

## Game mods and file access

Game addons can install or update mod files inside the user's game folder using the scoped [`files` API](./api-file-access.md) (`FILE_ACCESS` permission).

**Typical flow:** ask for the game folder in settings (`folder` / `file` field), then call `files.requestAccess(gamePath, 'manage')` and copy your mod files with `writeFile` / `copyFile`.

Example — Script Hook–style layout for GTA V:

```js
GenerateConfig([
  {
    key: 'game_dir',
    type: 'folder',
    default: '',
    editor: { label: { en: 'GTA V folder' }, required: true },
  },
  {
    key: 'mod_notice',
    type: 'info',
    editor: {
      description: {
        en: 'Anti-cheat may ban online play. Use only in single-player or allowed modded sessions.',
      },
      infoBorder: 'red',
    },
  },
]);

const params = await api.config.getParams();
const gameDir = params.game_dir;
if (gameDir && (await files.requestAccess(gameDir, 'manage')).success) {
  await files.writeFile(`${gameDir}\\scripts\\my_mod.asi`, '...');
}
```

### Developer responsibilities

- **Least privilege** — request only `read` or `manage` on the smallest path needed; never ask for whole-disk access.
- **Prefer established tools** — integrate via well-known mod loaders (e.g. Script Hook V for GTA V) or official mod APIs instead of custom injectors when possible.
- **Warnings** — if interaction can harm the PC, game account, or trigger anti-cheat, show a clear `info` block (`infoBorder: 'red'` or `yellow`) in settings.
- **Platform** — declare supported platforms in the manifest (`platforms`); game mods meant only for Windows must not break macOS/Linux users. Violations may lead to addon and author bans.
- **Prohibited games** — do not build addons for games that forbid this kind of integration; such addons and accounts will be blocked.
- **Closed source** — game modules may ship without source code, but catalog publication may be rejected if reviewers cannot verify safety without sources.

See [File access](./api-file-access.md) and [Settings schema](./settings-schema.md).

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
