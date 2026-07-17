# LLM Access (`llm`)

**Требуется:** `LLM`

Отправляет текстовые или диалоговые запросы через профили LLM пользователя (Ollama, Gemini, Groq, Cerebras, Puter и др.). API-ключи остаются в основном процессе и никогда не передаются аддонам.

## Настройка

1. Добавьте `"LLM"` в `permissions` в `manifest.json` (пользователь подтверждает при установке).
2. Проверьте в рантайме через `permissions.has(AddonsPermission.LLM)`.
3. При необходимости вызовите `llm.getSettings()` / `llm.listProfiles()` перед `llm.chat`.

```json
{
  "id": "my-addon",
  "permissions": ["LLM"]
}
```

## Как это работает

- Используются **те же** профили и маршрутизация, что в **Настройки → LLM Access**.
- `defaultProfile` может быть `auto` или конкретным id профиля.
- `options.profile` учитывается только если включено **Разрешить аддонам выбирать профиль**.
- `freeOnly` пропускает платные профили / похожие на платные модели.
- При rate limit (HTTP 429 / Retry-After) профиль уходит в cooldown, возможен fallback.

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

`input` — строка или массив `{ role, content }`. Опции: `profile`, `temperature`, `topP`, `maxTokens`, `freeOnly`, `timeoutMs` и др.

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

См. также [Permissions](./permissions.md) и [API overview](./api-overview.md).
