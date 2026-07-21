# Завантаження через yt-dlp (`ytdlp`)

**Потрібно:** `FILE_ACCESS`, `NETWORK_REQUEST`

Завантаження медіа з підтримуваних сайтів через **вбудований бінарник yt-dlp** у головному процесі. Аддон не запускає yt-dlp напряму.

## Налаштування

1. Додайте `"FILE_ACCESS"` і `"NETWORK_REQUEST"` до `permissions` у `manifest.json`.
2. Оберіть папку призначення:
   - **Папка користувача:** запросіть доступ **manage** (`files.requestAccess(folder, 'manage')`).
   - **Тимчасова папка аддона:** пишіть у `ADDON_TMP_DIR` — діалог згоди не потрібен.
3. Підпишіться на прогрес і викличте `ytdlp.downloadFile`.

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

## Як це працює

- Завантаження виконується в **головному процесі** через вбудований yt-dlp (`yt-dlp_linux`, `yt-dlp_macos`, `yt-dlp.exe`).
- Шлях збереження передається в yt-dlp `-o`. У імені файлу можна використовувати **змінні шаблону yt-dlp** (див. нижче).
- **Літеральні** (не шаблонні) частини шляху очищуються від недопустимих символів. Підставлені значення (наприклад `%(title)s`) зберігаються як є, включно з кирилицею.
- **Батьківська папка** вихідного шляху має бути покрита схваленим користувачем грантом **manage** (`files.requestAccess(folder, 'manage')`) **або** лежати всередині `ADDON_TMP_DIR`.
- Опції конвертації: `format` → `-f`, `extractAudio` → `-x`, `audioFormat` → `--audio-format`, `mergeOutputFormat` → `--merge-output-format`.
- Прогрес надходить у воркер аддона подією `ytdlp:download-progress`.
- Паралельні HLS/DASH-фрагменти (`concurrentFragments`) — від `1` до `10` (`--concurrent-fragments`).

## `ytdlp.downloadFile(url, outputPath, options?)`

| Параметр | Обов'язковий | Опис |
| --- | --- | --- |
| `url` | так | URL медіа (`http://` або `https://`) |
| `outputPath` | так | Абсолютний шлях; допускаються змінні на кшталт `%(title)s.%(ext)s` |
| `options.downloadId` | ні | Зв'язок із подіями прогресу; генерується автоматично |
| `options.concurrentFragments` | ні | Паралельні фрагменти, `1`…`10` |
| `options.format` | ні | Селектор `-f` yt-dlp (наприклад `ba/bestaudio`) |
| `options.extractAudio` | ні | При `true` передає `-x` (видобути аудіо) |
| `options.audioFormat` | ні | `--audio-format` разом із `extractAudio` (наприклад `m4a`) |
| `options.mergeOutputFormat` | ні | `--merge-output-format` (наприклад `mp4`) |

Повертає:

```js
{
  success: boolean,
  downloadId?: string,
  error?: YtDlpAddonErrorCode,
  message?: string,
}
```

### Прогрес (`ytdlp:download-progress`)

Підпишіться **до** виклику `downloadFile`:

```js
const downloadId = random.id();

events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;

  console.log(progress.stage, progress.percent, progress.speed, progress.eta);
  // progress.downloadedBytes, progress.totalBytes
});
```

`progress.stage` — `'downloading'` під час завантаження, `'done'` при успіху та `'cancelled'` при зупинці через `ytdlp.cancelDownload`.

## `ytdlp.cancelDownload(downloadId)`

Перериває активне завантаження з тим самим `downloadId`. Проміс `downloadFile` завершиться з `success: false` і `error: 'cancelled'`.

| Параметр | Обов'язковий | Опис |
| --- | --- | --- |
| `downloadId` | так | Той самий id, що передано в `downloadFile` (або повернутий ним) |

Повертає:

```js
{
  success: boolean,
  downloadId?: string,
  error?: 'no_permission' | 'not_found' | 'invalid_download_id',
  message?: string,
}
```

### Приклад скасування

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
  console.log('Завантаження зупинено користувачем');
}
```

### Приклад

```js
const outputDir = 'C:\\Videos\\clips';
await files.requestAccess(outputDir, 'manage');

const downloadId = random.id();
events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;
  status.Update({
    current: 'online',
    message: { uk: `Завантаження ${progress.percent.toFixed(1)}%` },
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

## Змінні шаблону виводу

У `outputPath` можна використовувати поля yt-dlp. Основні змінні:

### Ідентифікація

| Змінна | Опис |
| --- | --- |
| `%(id)s` | ID відео |
| `%(title)s` | Назва |
| `%(fulltitle)s` | Повна назва (без обрізання) |
| `%(description)s` | Опис |
| `%(webpage_url)s` | Посилання на сторінку |
| `%(original_url)s` | Вихідний URL |

### Формат

| Змінна | Опис |
| --- | --- |
| `%(ext)s` | Розширення (`mp4`, `mkv`, `webm`, …) |
| `%(format)s` | Обраний формат (рядок) |
| `%(format_id)s` | ID формату |
| `%(resolution)s` | Роздільність (наприклад `1920x1080`) |
| `%(height)s` | Висота |
| `%(width)s` | Ширина |
| `%(fps)s` | FPS |
| `%(vcodec)s` | Відеокодек |
| `%(acodec)s` | Аудіокодек |
| `%(tbr)s` | Загальний bitrate |

### Автор / канал

| Змінна | Опис |
| --- | --- |
| `%(uploader)s` | Ім'я каналу/автора |
| `%(uploader_id)s` | ID каналу |
| `%(uploader_url)s` | Посилання на канал |
| `%(channel)s` | Канал (YouTube/Twitch) |
| `%(channel_id)s` | ID каналу |
| `%(channel_url)s` | Посилання на канал |

### Дата й час

| Змінна | Опис |
| --- | --- |
| `%(upload_date)s` | Дата (`YYYYMMDD`) |
| `%(timestamp)s` | UNIX timestamp |
| `%(release_date)s` | Дата релізу (якщо є) |

### Тривалість

| Змінна | Опис |
| --- | --- |
| `%(duration)s` | Тривалість у секундах |
| `%(duration_string)s` | Зручний формат (наприклад `01:23:45`) |

### Статистика

| Змінна | Опис |
| --- | --- |
| `%(view_count)s` | Перегляди |
| `%(like_count)s` | Лайки |
| `%(comment_count)s` | Коментарі |
| `%(average_rating)s` | Рейтинг (рідко) |

### Метадані

| Змінна | Опис |
| --- | --- |
| `%(categories)s` | Категорії |
| `%(tags)s` | Теги (через кому) |
| `%(language)s` | Мова |
| `%(availability)s` | Доступність |

### Плейлисти

| Змінна | Опис |
| --- | --- |
| `%(playlist)s` | Ім'я плейлиста |
| `%(playlist_id)s` | ID плейлиста |
| `%(playlist_index)s` | Позиція |
| `%(playlist_title)s` | Назва плейлиста |

### Субтитри

| Змінна | Опис |
| --- | --- |
| `%(subtitles)s` | Субтитри |
| `%(automatic_captions)s` | Автосубтитри |

### Сайт / екстрактор

| Змінна | Опис |
| --- | --- |
| `%(extractor)s` | Сайт (`twitch`, `youtube`, …) |
| `%(extractor_key)s` | Ключ екстрактора |

### Twitch

| Змінна | Опис |
| --- | --- |
| `%(creator)s` | Автор кліпу |
| `%(is_live)s` | Прапорець трансляції |
| `%(start_time)s` | Початок сегмента |
| `%(end_time)s` | Кінець сегмента |

**Повний список:** [шаблон виводу yt-dlp](https://github.com/yt-dlp/yt-dlp#output-template)

## Коди помилок

При `success: false` поле `error`:

| Код | Причина |
| --- | --- |
| `no_permission` | Немає `FILE_ACCESS` і/або `NETWORK_REQUEST` у маніфесті |
| `no_file_access` | Папка виводу не покрита грантом **manage** — спочатку `files.requestAccess(folder, 'manage')` |
| `unsupported_platform` | Немає вбудованого yt-dlp для цієї ОС |
| `binary_missing` | Бінарник не знайдено в установці застосунку |
| `invalid_url` | Порожній або не `http(s)` URL |
| `invalid_path` | Порожній або невірний шлях виводу |
| `incorrect_url` | yt-dlp відхилив URL або не знайшов формати |
| `network_error` | Мережа/DNS/HTTP або помилка запуску процесу |
| `download_failed` | yt-dlp завершився з помилкою без точнішої класифікації |
| `cancelled` | Завантаження зупинено через `ytdlp.cancelDownload` |

Помилки `cancelDownload`:

| Код | Причина |
| --- | --- |
| `no_permission` | Немає `FILE_ACCESS` і/або `NETWORK_REQUEST` |
| `invalid_download_id` | Порожній `downloadId` |
| `not_found` | Немає активного завантаження з цим id (вже завершене або не запускалось) |

`message` — зрозумілий опис (часто stderr yt-dlp).

## Див. також

- [Доступ до файлів](./api-file-access.md)
- [Дозволи](./permissions.md)
- [Огляд API](./api-overview.md)
