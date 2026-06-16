# api.config & storage

## `api.config`

Усі методи async (IPC до main process).

| Method | Permission | Опис |
| --- | --- | --- |
| `getParams()` | — | Блок налаштувань цього аддона |
| `updateParams(data)` | — | Заміна params (з лімітом розміру) |
| `getConfig()` | `CONFIG_READ` | Повна конфігурація застосунку |
| `setConfig(data)` | `CONFIG_WRITE` | Патч конфігурації застосунку |
| `getAddonParams(addonId)` | `ADDON_CONFIG_READ` | Params іншого аддона |

### Приклади

```js
const params = await api.config.getParams();
await api.config.updateParams({ counter: (params.counter || 0) + 1 });

const other = await api.config.getAddonParams('other_addon');
if (other.success) {
  console.log(other.params);
}
```

`updateParams` повертає `{ success: true, params }` або `{ success: false, message }`.

## `api.openUrl(url)`

Відкриває URL у системному браузері за замовчуванням. Спеціальний дозвіл не потрібен.

## `api.restart()`

Перезапускає worker-процес цього аддона.

Не спрацює, якщо аддон **зупинено після повторних падінь** — користувач має перезапустити його в налаштуваннях аддона (↻). Див. [Життєвий цикл — Захист від циклу падінь](./lifecycle.md#захист-від-циклу-падінь).

## `storage`

Приватний JSON-файл за `{data.path}/storage_data.json`. **Дозвіл не потрібен.** Не показується в UI налаштувань.

| Method | Опис |
| --- | --- |
| `Read()` | Розбір файлу або `undefined`, якщо відсутній |
| `Write(data)` | Повна заміна файлу (merge вручну за потреби) |
| `Clear()` | Видалення файлу |

```js
const state = storage.Read() ?? { seen: [] };
state.seen.push(eventId);
storage.Write(state);
```

Значення мають бути JSON-серіалізованими. Файл видаляється при деінсталяції.

**Відмінність від `api.config`:** `storage` — для внутрішнього runtime-стану; `api.config` — для налаштувань користувача (і може з'являтися в UI налаштувань, коли поля мають `editor`).
