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

Оболонка застосунку завантажує ваш HTML у iframe. Main process формує URL iframe:

```
http://localhost:{WEB_SERVER_PORT}/addon_static/{id}/?token={data.token}&bg={hex}&dataTheme={themeId}
```

| Query-параметр | Опис |
| --- | --- |
| `token` | Токен доступу для встановлення (`data.token` у worker). Перевіряйте на приватних API-маршрутах. |
| `bg` | Колір фону активної теми (`--bg`), hex у URL-кодуванні (наприклад `%230e0e10` для `#0e0e10`). |
| `dataTheme` | Значення атрибута `data-theme` на `<html>`: `light`, `dark`, id кастомної теми (`custom-…`) або вбудованого пресета (`preset-…`). |

Той самий статичний маршрут, що й у віджетів. Id вікна: `addon-app:{addonId}`.

Читати ці query-параметри вручну **не обов'язково**, якщо ви підключаєте `/data/styles.css` — StreamKit+ автоматично вбудовує bootstrap-скрипт (див. [Теми](#теми) нижче).

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

Динамічний `<link>` або будь-яке згадування `/data/styles.css` у вихідному HTML (зокрема в inline-скриптах) теж враховується.

Вкажіть порт веб-сервера застосунку, якщо він інший (за замовчуванням `WEB_SERVER_PORT` = `3000`).

Примітки:

- Дозвіл аддона не потрібен — маршрут обслуговує основний застосунок, а не папка встановлення аддона.
- `/data/styles.css` включає вбудовані світлу/темну теми (у базовому бандлі), **усі** вбудовані пресети (`preset-…`) і збережені користувачем кастомні схеми. Файл перегенеровується при зміні кастомних тем у **Settings → Interface**.
- CSS-змінні теми збігаються з головним вікном (`--bg`, `--text`, `--btn-primary`, …). Перевизначення застосовуються за значенням `data-theme` на `<html>` (як у основному застосунку).
- Власні стилі аддона можна підключати після таблиці стилів і за потреби перевизначати правила.

## Теми

Коли StreamKit+ віддає HTML-сторінку **application**-аддона з посиланням на `/data/styles.css`, перед `</head>` вставляється невеликий inline bootstrap-скрипт. Він:

1. При завантаженні — читає `dataTheme` і `bg` з URL iframe і застосовує до `document.documentElement` (`data-theme`, `background`, `color-scheme`).
2. При зміні теми — слухає `postMessage` від оболонки застосунку (без перезавантаження iframe).
3. При зміні URL — знову читає query-параметри, якщо змінився `location.search`.

Копіювати цей скрипт в аддон не потрібно — достатньо підключити `/data/styles.css`, синхронізацію теми виконує оболонка.

### Значення `data-theme`

| Значення | Опис |
| --- | --- |
| `light` | Вбудована світла тема |
| `dark` | Вбудована темна тема |
| `custom-…` | Id користувацької схеми |
| `preset-…` | Id вбудованого пресета (наприклад `preset-twitch`) |

Якщо в налаштуваннях обрано тему **System**, `dataTheme` стає `light` або `dark` залежно від ОС.

### Оновлення на льоту (`postMessage`)

Коли користувач змінює тему (або ОС перемикає світлу/темну при обраній **System**), оболонка надсилає в iframe:

```json
{
  "type": "streamkit:theme",
  "dataTheme": "preset-twitch",
  "bg": "#0e0e10"
}
```

Вбудований bootstrap-скрипт застосовує payload автоматично. Щоб обробити зміну теми у власному JS (опційно):

```js
window.addEventListener('message', (event) => {
  const payload = event.data;
  if (!payload || payload.type !== 'streamkit:theme') return;
  console.log('Theme changed:', payload.dataTheme, payload.bg);
});
```

### Ручна обробка теми (просунутий варіант)

Якщо `/data/styles.css` **не** підключено, bootstrap-скрипт не вставляється і query-параметри теми не застосовуються автоматично. Їх можна прочитати при завантаженні:

```js
const params = new URLSearchParams(location.search);
const dataTheme = params.get('dataTheme');
const bg = params.get('bg');
if (dataTheme) document.documentElement.setAttribute('data-theme', dataTheme);
if (bg) document.documentElement.style.background = bg;
```

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
