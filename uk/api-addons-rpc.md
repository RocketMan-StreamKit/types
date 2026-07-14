# addons (RPC)

Комунікація між аддонами, окремо від `events` (HTTP). Main process маршрутизує запити між **увімкненими** worker.

## `addons.request(targetAddonId, method, params?)`

Викликає обробник, зареєстрований в іншому аддоні.

```js
const res = await addons.request('other_addon', 'getChannelId', { platform: 'example' });
if (res.success) {
  console.log(res.result);
}
```

Повертає `{ success: true, result? }` або `{ success: false, message? }`.

## `addons.onRequest(method, handler)`

Надає метод іншим аддонам.

```js
addons.onRequest('getChannelId', async ({ fromAddonId, params }) => {
  console.log('Request from', fromAddonId);
  return { channelId: '12345' };
});
```

Обробник отримує `{ fromAddonId, params }` і повертає payload відповіді.

## `addons.offRequest(method)`

Видаляє раніше зареєстрований обробник.

## `addons.getInfo(addonIds?)`

Читає маніфест і статус увімкнення встановлених аддонів. **Додатковий дозвіл не потрібен.**

Без `addonIds` повертаються всі встановлені аддони. Передайте масив id, щоб запитати конкретні аддони. Невідомі id включаються з `missing: true` та `enabled: false`.

```js
const all = await addons.getInfo();
if (all.success) {
  for (const addon of all.addons) {
    console.log(addon.id, addon.enabled, addon.manifest?.version);
  }
}

const pair = await addons.getInfo(['twitch', 'other_addon']);
```

Кожен запис: `{ id, enabled, manifest, missing }`. `manifest` — розібраний `manifest.json` або `null`, якщо файли відсутні.

## Залежності в маніфесті

Використовуйте `depends_on: ["other_addon"]` в `manifest.json`, коли ваш аддон потребує, щоб інший аддон був встановлений і увімкнений перед активацією.
