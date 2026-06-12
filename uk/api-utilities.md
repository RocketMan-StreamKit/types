# Утиліти

## `isDeveloperMode`

`true`, коли застосунок запущено без упаковки (dev-збірка) або користувач увімкнув **Developer mode** в налаштуваннях. Використовуйте для додаткового логування або debug-ендпоінтів; зберігайте звичайну поведінку, коли `false`. Зміна налаштування потребує перезапуску застосунку/worker.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
```

## `console`

`log`, `error`, `warn`, `info` — з префіксом `[Addon {id}]`.

## Таймери

| API | Примітки |
| --- | --- |
| `setTimeout(fn, delay, ...args)` | Затримка обмежена макс. 60 000 мс; помилки перехоплюються |
| `setInterval(fn, interval, ...args)` | Інтервал обмежений 50–60 000 мс |
| `clearTimeout` / `clearInterval` | Стандартні |
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

## `URL` і `URLSearchParams`

Безпечні для пісочниці реалізації для розбору та побудови URL у коді аддона.

## `require(name)`

Доступно лише з дозволом `ROOT`. Інакше кидає виняток.
