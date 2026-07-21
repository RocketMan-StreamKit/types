# addons (RPC і події)

Обмін між аддонами, окремо від `events` (HTTP). Головний процес маршрутизує запити та підписки на події між воркерами.

## `addons.request(targetAddonId, method, params?)`

Викликає обробник, зареєстрований в іншому **увімкненому та запущеному** аддоні.

```js
const res = await addons.request('other_addon', 'getChannelId', { platform: 'example' });
if (res.success) {
  console.log(res.result);
}
```

Повертає `{ success: true, result? }` або `{ success: false, message? }`.

## `addons.onRequest(method, handler)`

Експонує метод для інших аддонів.

```js
addons.onRequest('getChannelId', async ({ fromAddonId, params }) => {
  console.log('Request from', fromAddonId);
  return { channelId: '12345' };
});
```

Обробник отримує `{ fromAddonId, params }` і повертає payload відповіді.

## `addons.offRequest(method)`

Видаляє раніше зареєстрований обробник.

## `addons.emit(event, data?)`

Публікує подію всім аддонам, підписаним на цю подію у **поточного** аддона. Доставка fire-and-forget (обробники не відповідають емітеру).

```js
await addons.emit('streamOnline', { title: 'Live now' });
```

Повертає `{ success: true, delivered }` (`delivered` — кількість запущених підписників, яким пішла подія) або `{ success: false, message? }`.

## `addons.subscribe(sourceAddonId, event, handler)`

Підписка на події іншого **встановленого** аддона. Джерело може бути вимкнене або ще не запущене; події надходять, коли воно емітить у робочому стані.

```js
const sub = await addons.subscribe('twitch', 'streamOnline', ({ fromAddonId, data }) => {
  console.log('Event from', fromAddonId, data);
});
if (sub.success) {
  // later: sub.Destroy();
}
```

Обробник отримує `{ fromAddonId, data }`. Повертає `{ success: true, id, Destroy }` або `{ success: false, message? }`.

Викличте `Destroy()`, щоб відписатися. Допускається кілька підписок на одну пару source+event.

## `addons.getInfo(addonIds?)`

Читає маніфест і статус увімкнення встановлених аддонів. **Додатковий дозвіл не потрібен.**

Без `addonIds` повертаються всі встановлені аддони. Передайте масив id, щоб запросити конкретні аддони. Невідомі id включаються з `missing: true` і `enabled: false`.

```js
const all = await addons.getInfo();
if (all.success) {
  for (const addon of all.addons) {
    console.log(addon.id, addon.enabled, addon.manifest?.version);
  }
}

const pair = await addons.getInfo(['twitch', 'other_addon']);
```

Кожен запис: `{ id, enabled, manifest, missing }`. `manifest` — розібраний `manifest.json` або `null`, якщо файлів немає.

## Залежності в маніфесті

Використовуйте `depends_on: ["other_addon"]` у `manifest.json`, якщо аддону потрібно, щоб інший аддон був встановлений і увімкнений до активації.

## Коли використовувати RPC, а коли події

| Задача | API |
| --- | --- |
| Запросити дані / дію в іншого аддона і дочекатися відповіді | `request` / `onRequest` |
| Пушити оновлення, щоб інші не поліли | `emit` / `subscribe` |
