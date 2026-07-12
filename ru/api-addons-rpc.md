# addons (RPC)

Обмен между аддонами, отдельно от `events` (HTTP). Главный процесс маршрутизирует запросы между **включёнными** воркерами.

## `addons.request(targetAddonId, method, params?)`

Вызывает обработчик, зарегистрированный в другом аддоне.

```js
const res = await addons.request('other_addon', 'getChannelId', { platform: 'example' });
if (res.success) {
  console.log(res.result);
}
```

Возвращает `{ success: true, result? }` или `{ success: false, message? }`.

## `addons.onRequest(method, handler)`

Экспонирует метод для других аддонов.

```js
addons.onRequest('getChannelId', async ({ fromAddonId, params }) => {
  console.log('Request from', fromAddonId);
  return { channelId: '12345' };
});
```

Обработчик получает `{ fromAddonId, params }` и возвращает payload ответа.

## `addons.offRequest(method)`

Удаляет ранее зарегистрированный обработчик.

## `addons.getInfo(addonIds?)`

Читает манифест и статус включения установленных аддонов. **Дополнительное разрешение не требуется.**

Без `addonIds` возвращаются все установленные аддоны. Передайте массив id, чтобы запросить конкретные аддоны. Неизвестные id включаются с `missing: true` и `enabled: false`.

```js
const all = await addons.getInfo();
if (all.success) {
  for (const addon of all.addons) {
    console.log(addon.id, addon.enabled, addon.manifest?.version);
  }
}

const pair = await addons.getInfo(['twitch', 'other_addon']);
```

Каждая запись: `{ id, enabled, manifest, missing }`. `manifest` — разобранный `manifest.json` или `null`, если файлы отсутствуют.

## Зависимости в манифесте

Используйте `depends_on: ["other_addon"]` в `manifest.json`, когда вашему аддону нужно, чтобы другой аддон был установлен и включён до активации.
