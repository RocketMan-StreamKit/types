# Загрузки через yt-dlp (`ytdlp`)

**Требуется:** `FILE_ACCESS`, `NETWORK_REQUEST`

Загрузка медиа с поддерживаемых сайтов через **встроенный бинарник yt-dlp** в основном процессе. Аддон не запускает yt-dlp напрямую.

## Настройка

1. Добавьте `"FILE_ACCESS"` и `"NETWORK_REQUEST"` в `permissions` в `manifest.json`.
2. Выберите папку назначения:
   - **Папка пользователя:** запросите доступ **manage** (`files.requestAccess(folder, 'manage')`).
   - **Временная папка аддона:** пишите в `ADDON_TMP_DIR` — диалог согласия не нужен.
3. Подпишитесь на прогресс и вызовите `ytdlp.downloadFile`.

```js
const folder = 'C:\\Videos\\clips';
const access = await files.requestAccess(folder, 'manage');
if (!access.success) {
  console.warn(access.message);
  return;
}
```

```json
{
  "id": "my-addon",
  "permissions": ["FILE_ACCESS", "NETWORK_REQUEST"]
}
```

## Как это работает

- Загрузка выполняется в **основном процессе** через встроенный yt-dlp (`yt-dlp_linux`, `yt-dlp_macos`, `yt-dlp.exe`).
- Путь сохранения передаётся в yt-dlp `-o`. В имени файла можно использовать **переменные шаблона yt-dlp** (см. ниже).
- **Литеральные** (не шаблонные) части пути очищаются от недопустимых символов. Подставленные значения (например `%(title)s`) сохраняются как есть, включая кириллицу.
- **Родительская папка** выходного пути должна быть покрыта одобренным пользователем грантом **manage** (`files.requestAccess(folder, 'manage')`) **или** лежать внутри `ADDON_TMP_DIR`.
- Опции конвертации: `format` → `-f`, `extractAudio` → `-x`, `audioFormat` → `--audio-format`, `mergeOutputFormat` → `--merge-output-format`.
- Прогресс приходит в воркер аддона событием `ytdlp:download-progress`.
- Параллельные HLS/DASH-фрагменты (`concurrentFragments`) — от `1` до `10` (`--concurrent-fragments`).

## `ytdlp.downloadFile(url, outputPath, options?)`

| Параметр | Обязателен | Описание |
| --- | --- | --- |
| `url` | да | URL медиа (`http://` или `https://`) |
| `outputPath` | да | Абсолютный путь; допускаются переменные вроде `%(title)s.%(ext)s` |
| `options.downloadId` | нет | Связь с событиями прогресса; генерируется автоматически |
| `options.concurrentFragments` | нет | Параллельные фрагменты, `1`…`10` |
| `options.format` | нет | Селектор `-f` yt-dlp (например `ba/bestaudio`) |
| `options.extractAudio` | нет | При `true` передаёт `-x` (извлечь аудио) |
| `options.audioFormat` | нет | `--audio-format` вместе с `extractAudio` (например `m4a`) |
| `options.mergeOutputFormat` | нет | `--merge-output-format` (например `mp4`) |

Возвращает:

```js
{
  success: boolean,
  downloadId?: string,
  error?: YtDlpAddonErrorCode,
  message?: string,
}
```

### Прогресс (`ytdlp:download-progress`)

Подпишитесь **до** вызова `downloadFile`:

```js
const downloadId = random.id();

events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;

  console.log(progress.stage, progress.percent, progress.speed, progress.eta);
  // progress.downloadedBytes, progress.totalBytes
});
```

`progress.stage` — `'downloading'` во время загрузки, `'done'` при успехе и `'cancelled'` при остановке через `ytdlp.cancelDownload`.

## `ytdlp.cancelDownload(downloadId)`

Прерывает активную загрузку с тем же `downloadId`. Промис `downloadFile` завершится с `success: false` и `error: 'cancelled'`.

| Параметр | Обязателен | Описание |
| --- | --- | --- |
| `downloadId` | да | Тот же id, что передан в `downloadFile` (или возвращён им) |

Возвращает:

```js
{
  success: boolean,
  downloadId?: string,
  error?: 'no_permission' | 'not_found' | 'invalid_download_id',
  message?: string,
}
```

### Пример отмены

```js
const downloadId = random.id();
const downloadPromise = ytdlp.downloadFile(url, outputPath, { downloadId });

cancelButton.onClick(async () => {
  const cancelled = await ytdlp.cancelDownload(downloadId);
  if (!cancelled.success) {
    console.warn(cancelled.error, cancelled.message);
  }
});

const result = await downloadPromise;
if (result.error === 'cancelled') {
  console.log('Загрузка остановлена пользователем');
}
```

### Пример

```js
const outputDir = 'C:\\Videos\\clips';
await files.requestAccess(outputDir, 'manage');

const downloadId = random.id();
events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;
  status.Update({
    current: 'online',
    message: { ru: `Загрузка ${progress.percent.toFixed(1)}%` },
  });
});

const result = await ytdlp.downloadFile(
  'https://www.twitch.tv/videos/1234567890',
  `${outputDir}\\%(uploader)s - %(title)s.%(ext)s`,
  { downloadId, concurrentFragments: 4 }
);

if (!result.success) {
  console.warn(result.error, result.message);
}
```

## Переменные шаблона вывода

В `outputPath` можно использовать поля yt-dlp. Основные переменные:

### Идентификация

| Переменная | Описание |
| --- | --- |
| `%(id)s` | ID видео |
| `%(title)s` | Название |
| `%(fulltitle)s` | Полное название (без обрезки) |
| `%(description)s` | Описание |
| `%(webpage_url)s` | Ссылка на страницу |
| `%(original_url)s` | Исходный URL |

### Формат

| Переменная | Описание |
| --- | --- |
| `%(ext)s` | Расширение (`mp4`, `mkv`, `webm`, …) |
| `%(format)s` | Выбранный формат (строка) |
| `%(format_id)s` | ID формата |
| `%(resolution)s` | Разрешение (например `1920x1080`) |
| `%(height)s` | Высота |
| `%(width)s` | Ширина |
| `%(fps)s` | FPS |
| `%(vcodec)s` | Видеокодек |
| `%(acodec)s` | Аудиокодек |
| `%(tbr)s` | Общий bitrate |

### Автор / канал

| Переменная | Описание |
| --- | --- |
| `%(uploader)s` | Имя канала/автора |
| `%(uploader_id)s` | ID канала |
| `%(uploader_url)s` | Ссылка на канал |
| `%(channel)s` | Канал (YouTube/Twitch) |
| `%(channel_id)s` | ID канала |
| `%(channel_url)s` | Ссылка на канал |

### Дата и время

| Переменная | Описание |
| --- | --- |
| `%(upload_date)s` | Дата (`YYYYMMDD`) |
| `%(timestamp)s` | UNIX timestamp |
| `%(release_date)s` | Дата релиза (если есть) |

### Длительность

| Переменная | Описание |
| --- | --- |
| `%(duration)s` | Длительность в секундах |
| `%(duration_string)s` | Читаемый формат (например `01:23:45`) |

### Статистика

| Переменная | Описание |
| --- | --- |
| `%(view_count)s` | Просмотры |
| `%(like_count)s` | Лайки |
| `%(comment_count)s` | Комментарии |
| `%(average_rating)s` | Рейтинг (редко) |

### Метаданные

| Переменная | Описание |
| --- | --- |
| `%(categories)s` | Категории |
| `%(tags)s` | Теги (через запятую) |
| `%(language)s` | Язык |
| `%(availability)s` | Доступность |

### Плейлисты

| Переменная | Описание |
| --- | --- |
| `%(playlist)s` | Имя плейлиста |
| `%(playlist_id)s` | ID плейлиста |
| `%(playlist_index)s` | Позиция |
| `%(playlist_title)s` | Название плейлиста |

### Субтитры

| Переменная | Описание |
| --- | --- |
| `%(subtitles)s` | Субтитры |
| `%(automatic_captions)s` | Автосубтитры |

### Сайт / экстрактор

| Переменная | Описание |
| --- | --- |
| `%(extractor)s` | Сайт (`twitch`, `youtube`, …) |
| `%(extractor_key)s` | Ключ экстрактора |

### Twitch

| Переменная | Описание |
| --- | --- |
| `%(creator)s` | Автор клипа |
| `%(is_live)s` | Флаг трансляции |
| `%(start_time)s` | Начало сегмента |
| `%(end_time)s` | Конец сегмента |

**Полный список:** [шаблон вывода yt-dlp](https://github.com/yt-dlp/yt-dlp#output-template)

## Коды ошибок

При `success: false` поле `error`:

| Код | Причина |
| --- | --- |
| `no_permission` | Нет `FILE_ACCESS` и/или `NETWORK_REQUEST` в манифесте |
| `no_file_access` | Папка вывода не покрыта грантом **manage** — сначала `files.requestAccess(folder, 'manage')` |
| `unsupported_platform` | Нет встроенного yt-dlp для этой ОС |
| `binary_missing` | Бинарник не найден в установке приложения |
| `invalid_url` | Пустой или не `http(s)` URL |
| `invalid_path` | Пустой или неверный путь вывода |
| `incorrect_url` | yt-dlp отклонил URL или не нашёл форматы |
| `network_error` | Сеть/DNS/HTTP или ошибка запуска процесса |
| `download_failed` | yt-dlp завершился с ошибкой без более точной классификации |
| `cancelled` | Загрузка остановлена через `ytdlp.cancelDownload` |

Ошибки `cancelDownload`:

| Код | Причина |
| --- | --- |
| `no_permission` | Нет `FILE_ACCESS` и/или `NETWORK_REQUEST` |
| `invalid_download_id` | Пустой `downloadId` |
| `not_found` | Нет активной загрузки с этим id (уже завершена или не запускалась) |

`message` — читаемое описание (часто stderr yt-dlp).

## См. также

- [Доступ к файлам](./api-file-access.md)
- [Разрешения](./permissions.md)
- [Обзор API](./api-overview.md)
