# Application-аддони

Applications — web UI, що відкриваються з **головного вікна** в окремому sandboxed `BrowserWindow` (без Node integration, без preload).

## Manifest

| Поле | Значення |
| --- | --- |
| `type` | `application` |
| `web` | HTML entry file |
| `web_type` | `application` (обов'язково) |
| `web_contents` | Додаткові ресурси |

## URL запуску

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}
```

Той самий статичний маршрут, що й у віджетів. Id вікна: `addon-app:{addonId}`.

## Стилі як у основному застосунку

StreamKit+ віддає зібрану копію основної таблиці стилів UI через локальний веб-сервер. Підключіть її в HTML застосунку, щоб кнопки, поля вводу, типографіка та CSS-змінні теми збігалися з іншими вікнами застосунку.

| Ресурс | URL |
| --- | --- |
| Повний CSS | `http://localhost:{WEB_SERVER_PORT}/data/styles.css` |
| Шрифти | `http://localhost:{WEB_SERVER_PORT}/data/fonts/…` (підключаються з CSS) |

Додайте у HTML entry file **перед** власними стилями аддона:

```html
<link rel="stylesheet" href="http://localhost:3000/data/styles.css">
```

Вкажіть порт веб-сервера застосунку, якщо він інший (за замовчуванням `WEB_SERVER_PORT` = `3000`).

Примітки:

- Дозвіл аддона не потрібен — маршрут обслуговує основний застосунок, а не папка встановлення аддона.
- CSS-змінні теми збігаються з головним вікном (`--bg`, `--text`, `--btn-primary`, …). Темна тема вмикається атрибутом `data-theme="dark"` на `<html>` (як у основному застосунку).
- Власні стилі аддона можна підключати після цього `<link>` і за потреби перевизначати правила.

## Статичний application (без worker)

Лише `WEB_CONTENT` — HTML/JS/CSS у папці встановлення.

## Повний application (worker)

Worker надає API-ендпоінти для сторінки. Перевіряйте `query.token === data.token` на приватних маршрутах (той самий патерн, що у віджетів).

Типові ендпоінти:

- `GET params` — налаштування для сторінки
- `GET state` — поточний стан
- `POST …` — дії користувача

## UI головного вікна

Коли існує хоча б один **увімкнений** application-аддон з валідним `web`, головне вікно показує **Applications**. Користувач обирає пункт; main process відкриває або фокусує sandbox-вікно через IPC.

Список оновлюється при install/enable/disable/uninstall.

## UI налаштувань

**Settings → Applications**: `AddonsCategoryBlock` з `type="application"` — той самий потік install/enable/settings, що й в інших категоріях.
