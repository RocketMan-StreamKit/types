# Доступ к файлам (`files` API)

Требуется право `FILE_ACCESS` в манифесте. Прямой `require('fs')` недоступен (нужен `ROOT`). Используйте глобальный объект `files`: пользователь одобряет каждый путь; main process ограничивает область.

## Как это работает

1. Укажите `"FILE_ACCESS"` в `manifest.json`.
2. `files.requestAccess(абсолютныйПуть, 'read' | 'manage')` — нативный диалог согласия.
3. Доступ к папке включает все вложенные файлы и папки.
4. `manage` включает чтение и метаданные.
5. Два отказа подряд блокируют запрос на **30 секунд**.
6. События `fileAccessGranted` / `fileAccessRevoked` через `events.On`.
7. Пользователь может отозвать доступ в настройках аддона.

## Пример

```js
const result = await files.requestAccess('C:\\Games\\GTAV', 'manage');
if (result.success) {
  await files.writeFile('C:\\Games\\GTAV\\scripts\\mod.asi', '...');
}

events.On('fileAccessGranted', grant => console.log(grant.path));
events.On('fileAccessRevoked', grant => console.log('revoked', grant.path));
```

## Методы

`requestAccess`, `revokeAccess`, `hasAccess`, `listAccess`, `readFile`, `writeFile`, `appendFile`, `readdir`, `stat`, `exists`, `mkdir`, `unlink`, `rename`, `copyFile`.

### `rename` и `copyFile`

- Источник должен быть **строго внутри** выданной папки (не корень гранта).
- Саму выданную папку переименовать/скопировать нельзя.
- Путь назначения должен иметь **manage**-доступ (та же или другая одобренная папка).

Подробнее на английском: [api-file-access.md](../en/api-file-access.md).
