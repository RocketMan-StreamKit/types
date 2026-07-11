# Language detection (`language`)

Detects the language of arbitrary text using the same fastText model as TTS (`lid.176.ftz`, ~176 languages). **No permission is required.**

Unlike TTS auto-detect (which maps results to `en` / `ru` / `uk` only), this API returns the **full** ISO language code from the model (for example `de`, `fr`, `ja`, `pl`, `es`).

## How it works

- Uses fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (downloaded on first use into app user data).
- Returns ISO-639-1 (`language`) and, when available, ISO-639-3 (`alpha3`) plus an English reference `name`.
- When the model is unavailable, falls back to a Cyrillic script hint (`uk` when і/ї/є/ґ are present, otherwise `ru`). Other scripts without a loaded model cannot be classified.
- Empty / whitespace-only text fails with `{ success: false }`.

## `language.detect(text)`

| Parameter | Required | Description |
| --- | --- | --- |
| `text` | yes | Message or arbitrary string to classify |

```js
const res = await language.detect('Bonjour le monde!');
if (res.success) {
  console.log(res.language); // 'fr'
  console.log(res.alpha3);   // 'fra' (when provided)
  console.log(res.name);     // 'French' (when provided)
}

const ja = await language.detect('こんにちは');
// { success: true, language: 'ja', alpha3: 'jpn', name: 'Japanese' }

const empty = await language.detect('   ');
// { success: false, message: 'Text is empty' }
```

| Field | Description |
| --- | --- |
| `language` | ISO-639-1 two-letter code (e.g. `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | ISO-639-3 three-letter code when the model provides it |
| `name` | English reference name from the model when available |

Common `message` values when `success` is `false`:

| Message | Cause |
| --- | --- |
| `Text is empty` | Blank or whitespace-only `text` |
| `Language could not be detected` | Model unavailable / uncertain and no Cyrillic script hint |

## Example: route chat by language

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    return;
  }

  if (res.language === 'uk') {
    // handle Ukrainian
  } else if (res.language === 'ru') {
    // handle Russian
  } else if (res.language === 'en') {
    // handle English
  } else {
    console.log('Other language:', res.language, res.name);
  }
});
```

## Relation to TTS

- `tts.speak` still auto-detects only `en` / `ru` / `uk` (English fallback) when `options.language` is omitted.
- Use `language.detect` when you need the broader language set; pass an explicit `options.language` to `tts.speak` if you map the result yourself to a supported TTS voice.

See also [Text-to-speech](./api-tts.md), [Localization](./localization.md), and [API overview](./api-overview.md).
