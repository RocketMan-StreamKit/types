# Утилиты

## `isDeveloperMode`

`true`, когда приложение запущено без упаковки (dev-сборка) или пользователь включил **Режим разработчика** в настройках. Используйте для дополнительного логирования или отладочных эндпоинтов; при `false` сохраняйте обычное поведение. Смена настройки требует перезапуска приложения/воркера.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
```

## `console`

`log`, `error`, `warn`, `info` — с префиксом `[Addon {id}]`.

## Таймеры

| API | Примечания |
| --- | --- |
| `setTimeout(fn, delay, ...args)` | Задержка ограничена макс. 60 000 мс; ошибки перехватываются |
| `setInterval(fn, interval, ...args)` | Интервал ограничен 50–60 000 мс |
| `clearTimeout` / `clearInterval` | Стандартное поведение |
| `sleep(ms)` | Promise; макс. 60 000 мс |

## `random`

```js
const n = random.number(1, 6);
const id = random.id();
```

## `crypto`

```js
const { verifier, challenge } = crypto.createPkce();

const ok = crypto.verifyRsaSha256(publicKeyPem, message, signatureBase64);
```

## `URL` и `URLSearchParams`

Реализации, безопасные для песочницы, для разбора и сборки URL внутри кода аддона.

## `require(name)`

Доступен только с разрешением `ROOT`. Иначе выбрасывает исключение.
