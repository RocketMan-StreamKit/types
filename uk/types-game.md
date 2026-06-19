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
  "permissions": ["NETWORK_REQUEST", "FILE_ACCESS"]
}
```

## Ігрові моди та доступ до файлів

Ігрові аддони можуть встановлювати або оновлювати мод-файли в папці гри через API [`files`](./api-file-access.md) (право `FILE_ACCESS` у маніфесті).

**Типовий сценарій:** у налаштуваннях користувач вказує папку гри (`folder` / `file`), аддон викликає `files.requestAccess(шлях, 'manage')` і копіює файли через `writeFile` / `copyFile` — як Script Hook для GTA V.

```js
GenerateConfig([
  {
    key: 'game_dir',
    type: 'folder',
    default: '',
    editor: { label: { uk: 'Папка GTA V' }, required: true },
  },
  {
    key: 'mod_notice',
    type: 'info',
    editor: {
      description: {
        uk: 'Античит може заблокувати онлайн. Використовуйте лише в одиночній грі або дозволених сесіях.',
      },
      infoBorder: 'red',
    },
  },
]);
```

### Обов'язки розробника

- **Мінімальні права** — запитуйте лише `read` або `manage` на мінімально потрібний шлях.
- **Відомі рішення** — надавайте перевагу популярним мод-лоадерам (Script Hook V для GTA V) або офіційним API замість самописних інжекторів.
- **Попередження** — якщо мод може зашкодити ПК, акаунту або спрацювати античит, покажіть блок `info` з рамкою `red` / `yellow` у налаштуваннях.
- **Платформа** — вказуйте `platforms` у маніфесті; аддон лише для Windows не повинен ламатися на macOS/Linux. Порушення призводять до блокування аддона та автора.
- **Заборонені ігри** — не робіть інтеграції з іграми, де це прямо заборонено.
- **Закритий код** — допускається, але публікація в каталозі може бути відхилена без можливості перевірити вихідники.

Див. [Доступ до файлів](./api-file-access.md), [Схема налаштувань](./settings-schema.md).

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
