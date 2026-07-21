# Утиліти

## `api.getProcessStats`

Повертає навантаження CPU та RAM **цього worker-процесу аддона**. Додаткові дозволи не потрібні.

| Поле | Тип | Примітка |
| --- | --- | --- |
| `cpuPercent` | `number` | CPU від попереднього виклику (перший виклик — `0`) |
| `ramMb` | `number` | Resident memory у мегабайтах |
| `pid` | `number` | PID worker-процесу цього аддона |

Підходить для самодіагностики, оптимізації або автоматичного відновлення (наприклад, перезапуск при перевищенні ліміту пам'яті). Можна поєднувати з `api.restart()` або очищенням внутрішніх кешів.

```js
const MAX_RAM_MB = 512;

setInterval(() => {
  const stats = api.getProcessStats();
  console.log('Worker usage', stats);

  if (stats.ramMb > MAX_RAM_MB) {
    console.warn('Memory limit exceeded, restarting worker');
    api.restart();
  }
}, 10_000);
```

Ті самі метрики показуються в списках аддонів у налаштуваннях StreamKit+ (PID в UI — лише в режимі розробника).

## `isDeveloperMode`

`true`, коли застосунок запущено без упаковки (dev-збірка) або користувач увімкнув **Developer mode** в налаштуваннях. Використовуйте для додаткового логування або debug-ендпоінтів; зберігайте звичайну поведінку, коли `false`. Зміна налаштування потребує перезапуску застосунку/worker.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
```

## `ADDON_TMP_DIR`

Абсолютний шлях до scratch-папки аддона в temp ОС:

`{os.tmpdir()}/StreamKitPlusAddons/{addonId}`

Створюється автоматично під час старту worker. Якщо в `addonId` є `/` (наприклад `ORG/REPO`), утворюються вкладені папки. Шляхи всередині **не** потребують `files.requestAccess`. Використовуйте з `files.*` (`FILE_ACCESS`) або `ytdlp.downloadFile`. Обробник ендпоінта може повернути `{ file: pathInsideTmp }`, щоб віддати медіа у веб-UI аддона.

```js
const sep = ADDON_TMP_DIR.includes('\\') ? '\\' : '/';
const out = `${ADDON_TMP_DIR}${sep}clip.%(ext)s`;
await ytdlp.downloadFile(url, out, { format: 'ba/bestaudio', extractAudio: true, audioFormat: 'm4a' });
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
