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
- **Volume:** `options.volumeMultiplier` scales playback relative to the user's TTS volume (`0` = silent, `1` = full user volume). Values above `1` are clamped to `1` — addons cannot exceed the user's volume setting.
- **Queue:** the main-process TTS manager serializes overlapping `speak` calls (from addons and from the app).

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
| ElevenLabs / Windows errors | API quota, missing voice, platform unavailable, etc. |

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
