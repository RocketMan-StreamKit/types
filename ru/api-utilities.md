# Утилиты

## `api.getProcessStats`

Возвращает загрузку CPU и RAM **этого worker-процесса аддона**. Дополнительные права не нужны.

| Поле | Тип | Примечание |
| --- | --- | --- |
| `cpuPercent` | `number` | CPU с момента предыдущего вызова (первый вызов — `0`) |
| `ramMb` | `number` | Resident memory в мегабайтах |
| `pid` | `number` | PID worker-процесса этого аддона |

Подходит для самодиагностики, оптимизации или автоматического восстановления (например, перезапуск при превышении лимита памяти). Можно сочетать с `api.restart()` или очисткой внутренних кэшей.

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

Те же метрики отображаются в списках аддонов в настройках StreamKit+ (PID в UI — только в режиме разработчика).

## `isDeveloperMode`

`true`, когда приложение запущено без упаковки (dev-сборка) или пользователь включил **Режим разработчика** в настройках. Используйте для дополнительного логирования или отладочных эндпоинтов; при `false` сохраняйте обычное поведение. Смена настройки требует перезапуска приложения/воркера.

```js
if (isDeveloperMode) {
  console.log('Debug info', params);
}
```

## `ADDON_TMP_DIR`

Абсолютный путь к scratch-папке аддона в temp ОС:

`{os.tmpdir()}/StreamKitPlusAddons/{addonId}`

Создаётся автоматически при старте воркера. Если в `addonId` есть `/` (например `ORG/REPO`), получаются вложенные папки. Пути внутри **не** требуют `files.requestAccess`. Используйте с `files.*` (`FILE_ACCESS`) или `ytdlp.downloadFile`. Обработчик эндпоинта может вернуть `{ file: pathInsideTmp }`, чтобы отдать медиа в веб-UI аддона.

```js
const sep = ADDON_TMP_DIR.includes('\\') ? '\\' : '/';
const out = `${ADDON_TMP_DIR}${sep}clip.%(ext)s`;
await ytdlp.downloadFile(url, out, { format: 'ba/bestaudio', extractAudio: true, audioFormat: 'm4a' });
```

## `console`

`log`, `error`, `warn`, `info` — с префиксом `[Addon {id}]`.

## Таймеры

| API | Примечания |
| --- | --- |
| `setTimeout(fn, delay, ...args)` | Ошибки перехватываются; delay передаётся в Node как есть |
| `setInterval(fn, interval, ...args)` | Ошибки перехватываются; interval передаётся в Node как есть |
| `clearTimeout` / `clearInterval` | Стандартное поведение |
| `sleep(ms)` | Promise-задержка |

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
