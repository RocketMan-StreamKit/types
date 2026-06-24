# status, notify, ui

## status

**Требуется:** `STATUS`

```js
status.Update({
  current: 'online', // offline | connecting | online | error
  message: { en: 'Connected', ru: 'Подключено', uk: 'Підключено' },
});

status.OnClick(() => {
  api.openUrl('https://example.com/dashboard');
});
```

Кликабельность отражается в строке состояния главного окна, когда зарегистрирован `OnClick`.

## viewers

**Разрешение не требуется**

Передаёт количество зрителей на платформе в строку состояния главного окна (слева от платформы/версии). Клик по блоку открывает настройки отображения.

```js
viewers.Update({
  platform: 'twitch',
  count: 128,
});
```

| Поле | Описание |
| --- | --- |
| `platform` | Идентификатор платформы. По возможности совпадает с id из `dashboard.registerPlatform` (например `twitch`, `youtube`). |
| `count` | Неотрицательное целое — число зрителей. |

**Поведение:**

- В интерфейсе учитываются только аддоны, которые хотя бы раз вызвали `Update`.
- Отключение аддона удаляет его данные об онлайне.
- Рядом с цифрой показывается иконка аддона из манифеста; при наведении — изменение счётчика с прошлого обновления.
- Пользователь выбирает режим: по платформам, по платформам + суммарный (иконка приложения и сумма) или только суммарный. Опционально — inline-дельта (+N зелёным / −N красным). **Настройки → Интерфейс** — показывать ли нулевой онлайн.

Пример вместе с регистрацией платформы:

```js
await dashboard.registerPlatform({
  id: 'twitch',
  name: { en: 'Twitch', ru: 'Twitch', uk: 'Twitch' },
});

viewers.Update({ platform: 'twitch', count: viewerCount });
```

## notify

**Требуется:** `NOTIFY`

```js
notify.Send({
  id: `${data.id}_status`,
  type: 'success', // success | info | warning | error
  title: { en: 'My Addon' },
  message: { en: 'Connection restored' },
  temp: true, // сбрасывается при следующем запуске приложения
});
```

Если задан `id`, предыдущее уведомление с тем же id заменяется. Используйте стабильные id для обновлений состояния соединения.

```js
notify.Remove(`${data.id}_status`);
```

Удаляет уведомление только если его создал этот аддон. Уведомления других аддонов и основного процесса игнорируются.

## settings.notify

Всплывающие уведомления поверх окна настроек аддона (разрешение не требуется). Их можно отправлять только пока открыта панель настроек этого аддона.

```js
if (settings.isOpen && !settings.isNotifyBlocked) {
  settings.notify.Send({
    title: { en: 'Settings saved', ru: 'Настройки сохранены' },
    message: { en: 'Token stored', ru: 'Токен сохранен' },
    buttonText: { en: 'Close', ru: 'Закрыть' },
  });
}
```

Более 5 уведомлений за 10 секунд блокируют показ на 5 секунд. Состояние блокировки доступно в `settings.isNotifyBlocked`.

## ui.auth

Страницы результата OAuth на локальном веб-сервере:

```js
ui.auth.generateSuccess('Account linked');
ui.auth.generateFail('Access denied');
```

Возврат из HTTP-обработчика:

```js
return { redirect: ui.auth.generateSuccess() };
```

Опциональный query-параметр `message` показывается на странице.
