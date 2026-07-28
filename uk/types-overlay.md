# Оверлей-аддони

Оверлеї — короткочасні ефекти, що запускаються правилами, таймерами або подіями dashboard.

## Manifest

| Поле | Значення |
| --- | --- |
| `type` | `overlay` або `overlay.info` |
| `web_type` | `overlay` (коли встановлено `web`) |

Панелі `overlay.info` рендеряться після або під час основного ефекту, коли на батьківському оверлеї увімкнено `showInfo`.

## Варіанти

### Простий медіа-оверлей (без worker)

`manifest.overlay` з шляхами `audio` / `video`. Без `index.js`, без дозволів. Відтворення через вбудований simple player iframe.

Налаштування: гучність, loop, hide-at-end, тривалість (можуть бути обмежені в маніфесті).

### Статичний web-оверлей (без worker)

`web` + `web_type: "overlay"`, без `index.js`. Потрібні `WEB_CONTENT` і файли з `web_contents`.

Статичний URL: `/addon_static/{id}/`

Runtime-налаштування (гучність, тривалість) можна читати з:

```
GET /overlay/settings/{overlayId}?token=
```

Коли оболонка оверлею програє ефект, в iframe URL додається `display=app` (вікно оверлею застосунку) або `display=web` (спільне OBS-посилання оверлею). Пряме відкриття `/addon_static/{id}/` — без `display`:

```js
const display = new URLSearchParams(window.location.search).get('display');
// null → відкрито поза оболонкою оверлею
// 'app' → вікно оверлею застосунку
// 'web' → спільне OBS-посилання оверлею
```

### Повний оверлей-аддон (worker)

`type: "overlay"`, worker `index.js`, `WEB_CONTENT` для кастомного HTML-ефекту.

Платформені аддони з `DASHBOARD_EVENTS` можуть реєструвати тригери оверлею через `dashboard.registerTriggers()` і передавати `trigger` в `addRecord()`.

## Дозволи

| Патерн | Permissions |
| --- | --- |
| Лише статичний HTML | `WEB_CONTENT` |
| Worker + HTTP для сторінки | `WEB_CONTENT`, `WEB_END_POINTS`, часто `SOCKET_END_POINTS` |

Перевіряйте `data.token` на ендпоінтах, призначених лише для web-сторінки оверлею.
