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

## Зависимости в манифесте

Используйте `depends_on: ["other_addon"]` в `manifest.json`, когда вашему аддону нужно, чтобы другой аддон был установлен и включён до активации.
