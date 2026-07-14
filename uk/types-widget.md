# Віджет-аддони

Віджети — **постійні** web-сторінки (алерти, цілі, чат-оверлеї). Вони не прив'язані до черги тригерів оверлею.

## Manifest

| Поле | Значення |
| --- | --- |
| `type` | `widget` |
| `web` | HTML entry file |
| `web_type` | `widget` (обов'язково) |
| `web_contents` | Додаткові ресурси |

## Прямий URL

Завжди доступний, коли увімкнено:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}
```

Показується в налаштуваннях як **Widget URL** для OBS Browser Source. Query-параметр `token` — frontend access token аддона.

## Статичний віджет (без worker)

Лише `manifest.json`, іконка, HTML і `web_contents`. Дозвіл: `WEB_CONTENT`.

## Повний віджет (worker)

Worker надає HTTP або Socket.IO для live-даних; web-сторінка отримує дані з `/addon/{id}/…` або простору імен Socket.IO.

Захищайте приватні ендпоінти:

```js
if (query.token !== data.token) {
  return { error: 'Unauthorized' };
}
```

## Відображення в оболонці оверлею

`widgets[]` в конфігурації застосунку для кожного встановленого віджета:

| Поле | Призначення |
| --- | --- |
| `display.app` | Вбудовування у прозоре вікно оверлей-застосунку |
| `display.web` | Вбудовування в OBS overlay browser source shell |

Прямий URL працює незалежно від прапорців display.

Live sync: простір імен Socket.IO `/overlay`, подія `widget:sync` з `{ widgets: [{ widgetId, url }] }`.

Snapshot: `GET /widget/snapshot?display=app|web&token=`

## UI налаштувань

**Settings → Widgets**: встановлення, увімкнення, поля `GenerateConfig`, прапорці display, копіювання widget URL.
