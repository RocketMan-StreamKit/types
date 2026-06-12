# api.config и storage

## `api.config`

Все методы асинхронные (IPC в главный процесс).

| Метод | Разрешение | Описание |
| --- | --- | --- |
| `getParams()` | — | Блок настроек этого аддона |
| `updateParams(data)` | — | Замена params (с лимитом размера) |
| `getConfig()` | `CONFIG_READ` | Полный конфиг приложения |
| `setConfig(data)` | `CONFIG_WRITE` | Патч конфига приложения |
| `getAddonParams(addonId)` | `ADDON_CONFIG_READ` | Params другого аддона |

### Примеры

```js
const params = await api.config.getParams();
await api.config.updateParams({ counter: (params.counter || 0) + 1 });

const other = await api.config.getAddonParams('other_addon');
if (other.success) {
  console.log(other.params);
}
```

`updateParams` возвращает `{ success: true, params }` или `{ success: false, message }`.

## `api.openUrl(url)`

Открывает URL в системном браузере по умолчанию. Специальное разрешение не нужно.

## `api.restart()`

Перезапускает процесс воркера этого аддона.

## `storage`

Приватный JSON-файл по пути `{data.path}/storage_data.json`. **Разрешение не требуется.** Не показывается в UI настроек.

| Метод | Описание |
| --- | --- |
| `Read()` | Разбор файла или `undefined`, если файла нет |
| `Write(data)` | Полная замена файла (слияние делайте вручную при необходимости) |
| `Clear()` | Удаление файла |

```js
const state = storage.Read() ?? { seen: [] };
state.seen.push(eventId);
storage.Write(state);
```

Значения должны быть JSON-сериализуемыми. Файл удаляется при деинсталляции.

**Отличие от `api.config`:** `storage` — для внутреннего состояния рантайма; `api.config` — пользовательские настройки (и могут появиться в UI настроек, если у полей есть `editor`).
