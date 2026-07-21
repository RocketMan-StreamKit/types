# yt-dlp downloads (`ytdlp`)

**Requires:** `FILE_ACCESS`, `NETWORK_REQUEST`

Downloads media from supported sites using the **bundled yt-dlp binary** in the main process. Addons never spawn yt-dlp directly.

## Setup

1. Add `"FILE_ACCESS"` and `"NETWORK_REQUEST"` to `permissions` in `manifest.json`.
2. Choose an output folder:
   - **User folder:** request **manage** access before downloading (`files.requestAccess(folder, 'manage')`).
   - **Addon temp folder:** write under `ADDON_TMP_DIR` — no consent dialog is required.
3. Subscribe to progress, then call `ytdlp.downloadFile`.

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

## How it works

- Downloads run in the **main process** via the bundled yt-dlp binary (`yt-dlp_linux`, `yt-dlp_macos`, `yt-dlp.exe`).
- The output path is passed to yt-dlp `-o`. You may use **yt-dlp output template variables** in the filename (see below).
- **Literal** (non-template) parts of the path are sanitized to remove invalid filename characters. Substituted values (for example `%(title)s`) are kept as-is, including Cyrillic characters.
- The **parent directory** of the output path must be covered by a user-approved **manage** file-access grant (`files.requestAccess(folder, 'manage')`), **or** lie under `ADDON_TMP_DIR`.
- Optional conversion flags map to yt-dlp: `format` → `-f`, `extractAudio` → `-x`, `audioFormat` → `--audio-format`, `mergeOutputFormat` → `--merge-output-format`.
- Progress is pushed to the addon worker as `ytdlp:download-progress` events while the download runs.
- Up to **10** parallel HLS/DASH fragments can be requested with `concurrentFragments` (maps to `--concurrent-fragments`).

## `ytdlp.downloadFile(url, outputPath, options?)`

| Parameter | Required | Description |
| --- | --- | --- |
| `url` | yes | Source media URL (`http://` or `https://`) |
| `outputPath` | yes | Absolute output path; may include yt-dlp variables like `%(title)s.%(ext)s` |
| `options.downloadId` | no | Correlates progress events; auto-generated when omitted |
| `options.concurrentFragments` | no | Parallel fragments, `1`…`10` (default: yt-dlp default) |
| `options.format` | no | yt-dlp `-f` selector (for example `ba/bestaudio`) |
| `options.extractAudio` | no | When `true`, pass `-x` to extract audio |
| `options.audioFormat` | no | `--audio-format` with `extractAudio` (for example `m4a`, `mp3`) |
| `options.mergeOutputFormat` | no | `--merge-output-format` (for example `mp4`) |

Returns:

```js
{
  success: boolean,
  downloadId?: string,
  error?: YtDlpAddonErrorCode,
  message?: string,
}
```

### Progress (`ytdlp:download-progress`)

Subscribe **before** calling `downloadFile`:

```js
const downloadId = random.id();

events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;

  console.log(progress.stage, progress.percent, progress.speed, progress.eta);
  // progress.downloadedBytes, progress.totalBytes
});
```

`progress.stage` is `'downloading'` during the transfer, `'done'` on success, and `'cancelled'` when `ytdlp.cancelDownload` stops the process.

## `ytdlp.cancelDownload(downloadId)`

Stops an active download started with the same `downloadId`. The original `downloadFile` promise resolves with `success: false` and `error: 'cancelled'`.

| Parameter | Required | Description |
| --- | --- | --- |
| `downloadId` | yes | Same id passed to `downloadFile` (or returned by it) |

Returns:

```js
{
  success: boolean,
  downloadId?: string,
  error?: 'no_permission' | 'not_found' | 'invalid_download_id',
  message?: string,
}
```

### Cancel example

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
  console.log('Download stopped by user');
}
```

### Example

```js
const outputDir = 'C:\\Videos\\clips';
await files.requestAccess(outputDir, 'manage');

const downloadId = random.id();
events.On('ytdlp:download-progress', ({ downloadId: id, progress }) => {
  if (id !== downloadId) return;
  status.Update({
    current: 'online',
    message: { en: `Downloading ${progress.percent.toFixed(1)}%` },
  });
});

const result = await ytdlp.downloadFile(
  'https://www.twitch.tv/videos/1234567890',
  `${outputDir}\\%(uploader)s - %(title)s.%(ext)s`,
  {
    downloadId,
    concurrentFragments: 4,
    format: 'bv*[height<=720]+ba/b[height<=720]',
    mergeOutputFormat: 'mp4',
  }
);

if (!result.success) {
  console.warn(result.error, result.message);
}
```

### Audio-only into `ADDON_TMP_DIR`

```js
const sep = ADDON_TMP_DIR.includes('\\') ? '\\' : '/';
const outputPath = `${ADDON_TMP_DIR}${sep}track.%(ext)s`;

const result = await ytdlp.downloadFile(url, outputPath, {
  format: 'ba/bestaudio',
  extractAudio: true,
  audioFormat: 'm4a',
});
```

## Output template variables

You can embed yt-dlp fields in `outputPath`. Common variables:

### Identification

| Variable | Description |
| --- | --- |
| `%(id)s` | Video ID |
| `%(title)s` | Title |
| `%(fulltitle)s` | Full title (not truncated) |
| `%(description)s` | Description |
| `%(webpage_url)s` | Page URL |
| `%(original_url)s` | Original URL passed to yt-dlp |

### Format

| Variable | Description |
| --- | --- |
| `%(ext)s` | File extension (`mp4`, `mkv`, `webm`, …) |
| `%(format)s` | Selected format (string) |
| `%(format_id)s` | Format ID |
| `%(resolution)s` | Resolution (e.g. `1920x1080`) |
| `%(height)s` | Video height |
| `%(width)s` | Video width |
| `%(fps)s` | Frames per second |
| `%(vcodec)s` | Video codec |
| `%(acodec)s` | Audio codec |
| `%(tbr)s` | Total bitrate |

### Author / channel

| Variable | Description |
| --- | --- |
| `%(uploader)s` | Channel or author name |
| `%(uploader_id)s` | Channel ID |
| `%(uploader_url)s` | Channel URL |
| `%(channel)s` | Channel name (YouTube/Twitch) |
| `%(channel_id)s` | Channel ID |
| `%(channel_url)s` | Channel URL |

### Date and time

| Variable | Description |
| --- | --- |
| `%(upload_date)s` | Upload date (`YYYYMMDD`) |
| `%(timestamp)s` | UNIX timestamp |
| `%(release_date)s` | Release date (when available) |

### Duration

| Variable | Description |
| --- | --- |
| `%(duration)s` | Duration in seconds |
| `%(duration_string)s` | Human-readable duration (e.g. `01:23:45`) |

### Statistics

| Variable | Description |
| --- | --- |
| `%(view_count)s` | View count |
| `%(like_count)s` | Like count |
| `%(comment_count)s` | Comment count |
| `%(average_rating)s` | Average rating (rare) |

### Metadata

| Variable | Description |
| --- | --- |
| `%(categories)s` | Categories |
| `%(tags)s` | Tags (comma-separated) |
| `%(language)s` | Language |
| `%(availability)s` | Availability |

### Playlists

| Variable | Description |
| --- | --- |
| `%(playlist)s` | Playlist name |
| `%(playlist_id)s` | Playlist ID |
| `%(playlist_index)s` | Position in playlist |
| `%(playlist_title)s` | Playlist title |

### Subtitles

| Variable | Description |
| --- | --- |
| `%(subtitles)s` | Subtitles |
| `%(automatic_captions)s` | Auto-generated captions |

### Site / extractor

| Variable | Description |
| --- | --- |
| `%(extractor)s` | Site name (`twitch`, `youtube`, …) |
| `%(extractor_key)s` | Extractor key |

### Twitch-specific

| Variable | Description |
| --- | --- |
| `%(creator)s` | Clip creator |
| `%(is_live)s` | Live flag |
| `%(start_time)s` | Segment start |
| `%(end_time)s` | Segment end |

**Full list:** [yt-dlp output template](https://github.com/yt-dlp/yt-dlp#output-template)

## Error codes

When `success` is `false`, `error` is one of:

| Code | Cause |
| --- | --- |
| `no_permission` | Missing `FILE_ACCESS` and/or `NETWORK_REQUEST` in manifest or sandbox check |
| `no_file_access` | Output folder not covered by a **manage** grant — call `files.requestAccess(folder, 'manage')` first |
| `unsupported_platform` | No bundled yt-dlp binary for this OS |
| `binary_missing` | Bundled binary not found in the app install |
| `invalid_url` | Empty or non-`http(s)` URL |
| `invalid_path` | Empty or invalid output path |
| `incorrect_url` | yt-dlp rejected the URL or found no formats |
| `network_error` | Network/DNS/HTTP failure or yt-dlp process spawn error |
| `download_failed` | yt-dlp exited with an error not classified above |
| `cancelled` | Download was stopped via `ytdlp.cancelDownload` |

`cancelDownload` errors:

| Code | Cause |
| --- | --- |
| `no_permission` | Missing `FILE_ACCESS` and/or `NETWORK_REQUEST` |
| `invalid_download_id` | Empty `downloadId` |
| `not_found` | No active download with this id (already finished or never started) |

`message` contains a human-readable detail string when available (often yt-dlp stderr).

## Related

- [File access](./api-file-access.md) — `files.requestAccess`, grants, revoke
- [Permissions](./permissions.md)
- [API overview](./api-overview.md)
