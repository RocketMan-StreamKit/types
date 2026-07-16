# Language detection (`language`)

Detects the language of arbitrary text using the same fastText model as TTS (`lid.176.ftz`, ~176 languages). **No permission is required.**

Unlike TTS auto-detect (which maps results to `en` / `ru` / `uk` only), this API returns the **full** ISO language code from the model (for example `de`, `fr`, `ja`, `pl`, `es`).

## How it works

- Uses fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (downloaded on first use into app user data).
- **Certain:** top-label probability ≥ `0.4` and gap to 2nd place ≥ `0.25` → `{ success: true, language, alpha3?, name? }`.
- **Uncertain:** otherwise → `{ success: false, message, possibles }` with ranked candidates (no single `language`). The addon decides whether to translate.
- Empty / whitespace-only text → `{ success: false, message: 'Text is empty' }` (no `possibles`).
- If the model is unavailable → `{ success: false, message, possibles: [] }`.

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

const uncertain = await language.detect('test message');
if (!uncertain.success) {
  console.log(uncertain.message);
  // 'Language is difficult to determine'
  console.log(uncertain.possibles);
  // [{ language: 'hi', probability: 0.07 }, { language: 'az', probability: 0.05 }, …]
}

const empty = await language.detect('   ');
// { success: false, message: 'Text is empty' }
```

### Success fields

| Field | Description |
| --- | --- |
| `language` | ISO-639-1 two-letter code (e.g. `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | ISO-639-3 three-letter code when the model provides it |
| `name` | English reference name from the model when available |

### Failure fields

| Field | Description |
| --- | --- |
| `message` | Human-readable reason |
| `possibles` | Optional ranked list `{ language, probability }[]` (highest first, up to 5). Present when the model ran but was unsure; may be `[]` if the model failed; omitted for empty text |

Common `message` values when `success` is `false`:

| Message | Cause |
| --- | --- |
| `Text is empty` | Blank or whitespace-only `text` |
| `Language is difficult to determine` | Low confidence, top languages too close, or model unavailable |

## Example: translate only when certain

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    // Optional: inspect res.possibles and decide manually
    // e.g. if (res.possibles?.[0]?.probability > 0.3) …
    return;
  }

  if (res.language === 'uk' || res.language === 'ru') {
    // translate…
  }
});
```

## Relation to TTS

- `tts.speak` still auto-detects only `en` / `ru` / `uk` (English / Cyrillic script fallback) when `options.language` is omitted.
- Use `language.detect` when you need the broader language set; pass an explicit `options.language` to `tts.speak` if you map the result yourself to a supported TTS voice.

See also [Text-to-speech](./api-tts.md), [Localization](./localization.md), and [API overview](./api-overview.md).
