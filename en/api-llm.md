# LLM Access (`llm`)

**Requires:** `LLM`

Sends text or multi-turn dialog requests through the user's configured LLM profiles (Ollama, Gemini, Groq, Cerebras, Puter, and other providers). API keys stay in the main process and are never exposed to addons.

## Setup

1. Add `"LLM"` to `permissions` in `manifest.json` (the user approves it at install time).
2. Check at runtime with `permissions.has(AddonsPermission.LLM)`.
3. Optionally call `llm.getSettings()` / `llm.listProfiles()` before `llm.chat`.

```json
{
  "id": "my-addon",
  "permissions": ["LLM"]
}
```

## How it works

- Uses the **same** profiles and routing as **Settings → LLM Access**.
- `defaultProfile` may be `auto` (host picks among enabled free-capable profiles) or a specific profile id.
- `options.profile` is honored only when the user enabled **Allow addons to choose a profile**.
- `freeOnly` (global or per-request) skips paid profiles / paid-looking models.
- On rate limits (HTTP 429 / Retry-After), the host cools down that profile and may fall back to another.

## `llm.listProfiles()`

Returns public profile metadata (no API keys), including soft quota snapshots when tracked.

```js
const { success, profiles, message } = await llm.listProfiles();
// profiles[].id, name, provider, model, params, enabled, isPaid,
// hasApiKey, contextWindow?, quota?
```

## `llm.getSettings()`

```js
const {
  success,
  enabled,
  defaultProfile,
  allowAddonProfileSelect,
  freeOnly,
  message,
} = await llm.getSettings();
```

| Field | Description |
| --- | --- |
| `enabled` | Master switch; when `false`, `chat` fails |
| `defaultProfile` | `'auto'` or a profile id |
| `allowAddonProfileSelect` | Whether `options.profile` overrides are allowed |
| `freeOnly` | Prefer / restrict to free profiles |

## `llm.chat(input, options?)`

| Parameter | Required | Description |
| --- | --- | --- |
| `input` | yes | Plain string **or** `{ role, content }[]` (`system` / `user` / `assistant`) |
| `options.profile` | no | Profile id or `'auto'` (needs `allowAddonProfileSelect` when overriding default) |
| `options.temperature` | no | Override profile default |
| `options.topP` | no | Override |
| `options.maxTokens` | no | Override |
| `options.frequencyPenalty` | no | Override |
| `options.presencePenalty` | no | Override |
| `options.stop` | no | Stop sequences |
| `options.freeOnly` | no | Force free-only for this call |
| `options.timeoutMs` | no | Request timeout |

```js
await llm.chat('Summarize this donation message');

await llm.chat(
  [
    { role: 'system', content: 'Reply briefly.' },
    { role: 'user', content: 'Hello' },
  ],
  { profile: 'auto', temperature: 0.4, maxTokens: 256 }
);
```

Returns:

```js
{
  success: boolean,
  content?: string,
  profileId?: string,
  provider?: string,
  model?: string,
  usage?: { promptTokens?, completionTokens?, totalTokens? },
  message?: string,
  triedProfiles?: string[],
}
```

## Example

```js
if (!permissions.has(AddonsPermission.LLM)) {
  console.warn('LLM not granted');
  return;
}

const { enabled } = await llm.getSettings();
if (!enabled) return;

const res = await llm.chat('Write a short thank-you for a donation');
if (res.success) {
  console.log(res.content);
}
```

See also [Permissions](./permissions.md) and [API overview](./api-overview.md).
