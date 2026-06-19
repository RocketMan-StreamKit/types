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
  "permissions": ["NETWORK_REQUEST", "FILE_ACCESS"]
}
```

## Игровые моды и доступ к файлам

Игровые аддоны могут устанавливать или обновлять мод-файлы в папке игры через API [`files`](./api-file-access.md) (право `FILE_ACCESS` в манифесте).

**Типичный сценарий:** в настройках пользователь указывает папку игры (`folder` / `file`), аддон вызывает `files.requestAccess(путь, 'manage')` и копирует свои файлы через `writeFile` / `copyFile` — как Script Hook для GTA V.

```js
GenerateConfig([
  {
    key: 'game_dir',
    type: 'folder',
    default: '',
    editor: { label: { ru: 'Папка GTA V' }, required: true },
  },
  {
    key: 'mod_notice',
    type: 'info',
    editor: {
      description: {
        ru: 'Античит может заблокировать онлайн. Используйте только в одиночной игре или разрешённых сессиях.',
      },
      infoBorder: 'red',
    },
  },
]);
```

### Обязанности разработчика

- **Минимальные права** — запрашивайте только `read` или `manage` на минимально нужный путь.
- **Известные решения** — предпочитайте популярные мод-лоадеры (Script Hook V для GTA V) или официальные API вместо самописных инжекторов.
- **Предупреждения** — если мод может навредить ПК, аккаунту или сработать античит, выведите блок `info` с рамкой `red` / `yellow` в настройках.
- **Платформа** — указывайте `platforms` в манифесте; аддон только под Windows не должен ломаться на macOS/Linux. Нарушения ведут к блокировке аддона и автора.
- **Запрещённые игры** — не делайте интеграции с играми, где это прямо запрещено.
- **Закрытый код** — допускается, но публикация в каталоге может быть отклонена без возможности проверить исходники.

См. [Доступ к файлам](./api-file-access.md), [Схема настроек](./settings-schema.md).

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
