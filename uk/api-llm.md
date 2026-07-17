# LLM Access (`llm`)

**Потрібно:** `LLM`

Надсилає текстові або діалогові запити через профілі LLM користувача (Ollama, Gemini, Groq, Cerebras, Puter тощо). API-ключі залишаються в основному процесі і ніколи не передаються аддонам.

## Налаштування

1. Додайте `"LLM"` до `permissions` у `manifest.json` (користувач підтверджує під час встановлення).
2. Перевірте в рантаймі через `permissions.has(AddonsPermission.LLM)`.
3. За потреби викличте `llm.getSettings()` / `llm.listProfiles()` перед `llm.chat`.

```json
{
  "id": "my-addon",
  "permissions": ["LLM"]
}
```

## Як це працює

- Використовуються **ті самі** профілі та маршрутизація, що в **Налаштування → LLM Access**.
- `defaultProfile` може бути `auto` або конкретним id профілю.
- `options.profile` враховується лише якщо увімкнено дозвіл аддонам обирати профіль.
- `freeOnly` пропускає платні профілі / схожі на платні моделі.
- При rate limit (HTTP 429 / Retry-After) профіль йде в cooldown, можливий fallback.

## `llm.listProfiles()`

```js
const { success, profiles, message } = await llm.listProfiles();
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

## `llm.chat(input, options?)`

`input` — рядок або масив `{ role, content }`. Опції: `profile`, `temperature`, `topP`, `maxTokens`, `freeOnly`, `timeoutMs` тощо.

```js
await llm.chat('Hello');
await llm.chat(
  [
    { role: 'system', content: 'Reply briefly.' },
    { role: 'user', content: 'Hi' },
  ],
  { profile: 'auto', temperature: 0.4 }
);
```

Див. також [Permissions](./permissions.md) та [API overview](./api-overview.md).
