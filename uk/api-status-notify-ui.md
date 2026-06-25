# status, notify, ui

## status

**Потрібно:** `STATUS`

```js
status.Update({
  current: 'online', // offline | connecting | online | error
  message: { en: 'Connected', ru: 'Подключено', uk: 'Підключено' },
});

status.OnClick(() => {
  api.openUrl('https://example.com/dashboard');
});
```

Клікабельність відображається в рядку стану головного вікна, коли зареєстровано `OnClick`.

## viewers

**Дозвіл не потрібен**

Передає кількість глядачів на платформі в рядок стану головного вікна (ліворуч від платформи/версії). Клік по блоку відкриває налаштування відображення.

```js
viewers.Update({
  platform: 'twitch',
  count: 128,
});
```

| Поле | Опис |
| --- | --- |
| `platform` | Ідентифікатор платформи. За можливості збігається з id з `dashboard.registerPlatform` (наприклад `twitch`, `youtube`). |
| `count` | Невід'ємне ціле — кількість глядачів, або `-1`, якщо стрім/платформа офлайн. |

**Поведінка:**

- В інтерфейсі враховуються лише аддони, які хоча б раз викликали `Update`.
- Вимкнення аддона видаляє його дані про онлайн.
- Поруч із цифрою показується іконка аддона з маніфесту; при наведенні — зміна лічильника з минулого оновлення.
- Користувач обирає режим: за платформами, за платформами + сумарний (іконка застосунку та сума) або лише сумарний. Опційно — inline-дельта (+N зеленим / −N червоним). **Налаштування → Інтерфейс** — показувати нульовий онлайн чи ні.

Приклад разом із реєстрацією платформи:

```js
await dashboard.registerPlatform({
  id: 'twitch',
  name: { en: 'Twitch', ru: 'Twitch', uk: 'Twitch' },
});

viewers.Update({ platform: 'twitch', count: viewerCount });
```

## notify

**Потрібно:** `NOTIFY`

```js
notify.Send({
  id: `${data.id}_status`,
  type: 'success', // success | info | warning | error
  title: { en: 'My Addon' },
  message: { en: 'Connection restored' },
  temp: true, // cleared on next app start
});
```

Коли встановлено `id`, попереднє сповіщення з тим самим id замінюється. Використовуйте стабільні id для оновлень стану з'єднання.

```js
notify.Remove(`${data.id}_status`);
```

Видаляє сповіщення лише якщо його створив цей аддон. Сповіщення інших аддонів і головного процесу ігноруються.

## settings.notify

Попап-сповіщення поверх вікна налаштувань аддона (дозвіл не потрібен). Їх можна надсилати лише коли відкрита панель налаштувань цього аддона.

```js
if (settings.isOpen && !settings.isNotifyBlocked) {
  settings.notify.Send({
    title: { en: 'Settings saved', uk: 'Налаштування збережено' },
    message: { en: 'Token stored', uk: 'Токен збережено' },
    buttonText: { en: 'Close', uk: 'Закрити' },
  });
}
```

Понад 5 сповіщень за 10 секунд блокують показ на 5 секунд. Стан блокування доступний у `settings.isNotifyBlocked`.

## ui.auth

Сторінки результату OAuth на локальному web-сервері:

```js
ui.auth.generateSuccess('Account linked');
ui.auth.generateFail('Access denied');
```

Поверніть з HTTP-обробника:

```js
return { redirect: ui.auth.generateSuccess() };
```

Необов'язковий query-параметр `message` показується на сторінці.
