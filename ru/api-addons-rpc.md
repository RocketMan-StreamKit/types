# addons (RPC и события)

Обмен между аддонами, отдельно от `events` (HTTP). Главный процесс маршрутизирует запросы и подписки на события между воркерами.

## `addons.request(targetAddonId, method, params?)`

Вызывает обработчик, зарегистрированный в другом **включённом и запущенном** аддоне.

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

## `addons.emit(event, data?)`

Публикует событие всем аддонам, подписанным на это событие у **текущего** аддона. Доставка fire-and-forget (обработчики не отвечают эмиттеру).

```js
await addons.emit('streamOnline', { title: 'Live now' });
```

Возвращает `{ success: true, delivered }` (`delivered` — число запущенных подписчиков, которым ушло событие) или `{ success: false, message? }`.

## `addons.subscribe(sourceAddonId, event, handler)`

Подписка на события другого **установленного** аддона. Источник может быть выключен или ещё не запущен; события приходят, когда он эмитит в рабочем состоянии.

```js
const sub = await addons.subscribe('twitch', 'streamOnline', ({ fromAddonId, data }) => {
  console.log('Event from', fromAddonId, data);
});
if (sub.success) {
  // later: sub.Destroy();
}
```

Обработчик получает `{ fromAddonId, data }`. Возвращает `{ success: true, id, Destroy }` или `{ success: false, message? }`.

Вызовите `Destroy()`, чтобы отписаться. Допускается несколько подписок на одну пару source+event.

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

Каждая запись: `{ id, enabled, manifest, missing }`. `manifest` — разобранный `manifest.json` или `null`, если файлов нет.

## Зависимости в манифесте

Используйте `depends_on: ["other_addon"]` в `manifest.json`, если аддону нужно, чтобы другой аддон был установлен и включён до активации.

## Когда использовать RPC, а когда события

| Задача | API |
| --- | --- |
| Запросить данные / действие у другого аддона и дождаться ответа | `request` / `onRequest` |
| Пушить обновления, чтобы другие не поллили | `emit` / `subscribe` |
