# Доступ до файлів (`files` API)

Потрібне право `FILE_ACCESS` у маніфесті. Прямий `require('fs')` недоступний (потрібен `ROOT`). Використовуйте глобальний об'єкт `files`: користувач схвалює кожен шлях; main process обмежує область.

## Як це працює

1. Вкажіть `"FILE_ACCESS"` у `manifest.json`.
2. `files.requestAccess(абсолютнийШлях, 'read' | 'manage')` — нативний діалог згоди.
3. Доступ до папки включає усі вкладені файли та папки.
4. `manage` включає читання та метадані.
5. Два відмови поспіль блокують запит на **30 секунд**.
6. Події `fileAccessGranted` / `fileAccessRevoked` через `events.On`.
7. Користувач може відкликати доступ у налаштуваннях аддона.

## Приклад

```js
const result = await files.requestAccess('C:\\Games\\GTAV', 'manage');
if (result.success) {
  await files.writeFile('C:\\Games\\GTAV\\scripts\\mod.asi', '...');
}

events.On('fileAccessGranted', grant => console.log(grant.path));
events.On('fileAccessRevoked', grant => console.log('revoked', grant.path));
```

## Методи

`requestAccess`, `revokeAccess`, `hasAccess`, `listAccess`, `readFile`, `writeFile`, `appendFile`, `readdir`, `stat`, `exists`, `mkdir`, `unlink`, `rename`, `copyFile`.

### `rename` та `copyFile`

- Джерело має бути **суворо всередині** наданої папки (не корінь гранта).
- Саму надану папку перейменувати/скопіювати не можна.
- Шлях призначення повинен мати **manage**-доступ (та сама або інша схвалена папка).

Детальніше англійською: [api-file-access.md](../en/api-file-access.md).
