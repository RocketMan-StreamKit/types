# Text-to-speech (`tts`)

**Requires:** `TTS`

Plays synthesized speech through the user's configured TTS engine (Piper, ElevenLabs, or Windows SAPI). Respects the master TTS enable switch, voice settings, and playback volume from the app settings UI.

## Setup

1. Add `"TTS"` to `permissions` in `manifest.json` (the user approves it at install time).
2. Check at runtime with `permissions.has(AddonsPermission.TTS)`.
3. Call `tts.getEngine()` to see whether TTS is enabled and which engine is active.

```json
{
  "id": "my-addon",
  "permissions": ["TTS", "DASHBOARD_EVENTS"]
}
```

## How it works

- Uses the **same** TTS configuration as the built-in Text to speech settings (engine, voices, enable switch, volume).
- Audio plays in the **main window** (same playback path as internal `speakWithTTS`).
- **Language:** pass `options.language` (`en`, `ru`, `uk`) or omit it — fastText auto-detects uk/ru/en (English fallback).
- For the **full** language set (~176 ISO codes), use [`language.detect`](./api-language.md) separately (no TTS permission). TTS itself still only uses `en` / `ru` / `uk`.
- **Volume:** `options.volumeMultiplier` scales playback relative to the user's TTS volume (`0` = silent, `1` = full user volume). Values above `1` are clamped to `1` — addons cannot exceed the user's volume setting.
- **Queue:** the main-process TTS manager serializes overlapping `speak` / `prepare` synthesis calls (from addons and from the app). `playPrepared` only plays an already-synthesized clip.

## `tts.getEngine()`

Returns the active engine and whether TTS playback is enabled in settings.

```js
const { success, engine, enabled, message } = await tts.getEngine();
// success: boolean
// engine: 'piper' | 'elevenlabs' | 'windows'  (when success)
// enabled: boolean                             (when success)
// message: string                              (when !success)
```

| Field | Description |
| --- | --- |
| `engine` | Active synthesis backend selected in user settings |
| `enabled` | `false` when the user disabled TTS globally — `speak` will return `{ success: false }` |

## `tts.getVoiceInfo(language?)`

Returns the **active voice summary** for the configured engine so addons can adapt LLM prompts (spoken gender / character) to the voice the streamer selected.

```js
const {
  success,
  engine,
  enabled,
  voiceName,
  voiceId,
  gender,
  language,
  message,
} = await tts.getVoiceInfo('ru');
```

| Field | Description |
| --- | --- |
| `engine` / `enabled` | Same meaning as `getEngine()` |
| `voiceName` | Human-readable voice / model name when known (`null` if unset) |
| `voiceId` | ElevenLabs voice id, or Piper model key; otherwise `null` |
| `gender` | `male` / `female` / `neutral` when the engine exposes it; otherwise `null` (infer from `voiceName` if needed) |
| `language` | Language key used for Piper / Windows voice selection; `null` for ElevenLabs (shared voice) |

Optional `language` (`en` / `ru` / `uk`) picks which Piper / Windows per-language voice to describe; when omitted, the host uses the app UI language (falling back to the first configured voice).

## `tts.speak(text, options?)`

| Parameter | Required | Description |
| --- | --- | --- |
| `text` | yes | Message to speak (trimmed; empty text fails) |
| `options.language` | no | `en`, `ru`, or `uk`. When omitted, language is auto-detected |
| `options.volumeMultiplier` | no | `0`…`1` relative to the user's TTS volume. Values above `1` are clamped to `1` |

```js
await tts.speak('Thanks for the follow!');

await tts.speak('Спасибо за донат!', { language: 'ru' });

await tts.speak('Quiet notification', { volumeMultiplier: 0.5 });
```

Returns `{ success: boolean, message?: string }`.

Common `message` values when `success` is `false`:

| Message | Cause |
| --- | --- |
| `TTS is disabled` | User turned off TTS in settings |
| `Text is empty` | Blank or whitespace-only `text` |
| `No TTS model configured for this language` | Piper engine, no model for detected language |
| `Permission TTS is required…` | Missing manifest permission (sandbox-side check) |
| `Prepared TTS clip not found or expired` | `playPrepared` / `discardPrepared` with unknown or expired id |
| ElevenLabs / Windows errors | API quota, missing voice, platform unavailable, etc. |

## `tts.prepare(text, options?)` / `tts.playPrepared(id, options?)` / `tts.discardPrepared(id)`

Synthesize speech **ahead of playback** so work (donation delay, LLM, etc.) can overlap with API / Piper synthesis. Prepared clips live in the main process for **10 minutes**, then expire.

| Method | Description |
| --- | --- |
| `prepare(text, options?)` | Synthesize only → `{ success, id?, message? }`. `options.language` same as `speak` |
| `playPrepared(id, options?)` | Play once and consume `id`. Optional `volumeMultiplier` like `speak` |
| `discardPrepared(id)` | Drop without playing |

```js
const delayMs = 5000;
const delayDone = new Promise(resolve => setTimeout(resolve, delayMs));

const reply = 'Thanks for the donation!';
const prepared = await tts.prepare(reply, { language: 'en' });

await delayDone;

if (prepared.success && prepared.id) {
  await tts.playPrepared(prepared.id);
} else {
  await tts.speak(reply, { language: 'en' });
}
```

**Queue notes:** `prepare` and `speak` share the same synthesis queue (no overlapping Piper / API jobs). `playPrepared` only starts playback and does not re-synthesize.

## Example: speak on dashboard event

```js
if (!permissions.has(AddonsPermission.TTS)) {
  console.warn('TTS not granted');
  return;
}

events.On('onDonation', async payload => {
  const { success, enabled } = await tts.getEngine();
  if (!success || !enabled) {
    return;
  }

  const username = payload.body?.username || 'viewer';
  await tts.speak(`Thank you ${username} for the donation!`, {
    volumeMultiplier: 0.8,
  });
});
```

See also [Permissions](./permissions.md) and [API overview](./api-overview.md).
