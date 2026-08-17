# LLM Access

**Settings → LLM Access** stores **profiles** for language models: Ollama on your PC, Gemini, Groq, OpenAI, OpenRouter, and other OpenAI-compatible providers.

Addons with permission can send chat requests through these profiles. **API keys never leave StreamKit+** and are not visible to addons.

You do not need this section unless an addon (or the built-in test chat) should talk to a model.

## Global options

| Setting | What it does |
| --- | --- |
| **Enable LLM Access** | When off, addons cannot call the model. The test chat on this page still works. |
| **Default profile** | **Auto** (best available enabled profile) or a specific profile |
| **Allow addons to choose a profile** | When off, addons always use the default selection |
| **Free-only mode** | Auto-routing skips profiles and models marked as paid |

Save the main settings bar after changing these options or editing profiles.

## Profiles

Add, edit, or delete a profile:

- Provider, model, optional base URL, API key
- Creativity (temperature) and max response length
- Enabled / paid flags

**Refresh models** loads the provider’s model list when supported. **Detect Ollama** looks for a local Ollama install.

If a provider hits a rate limit, auto mode can fall back to another enabled profile.

## Test chat

**Open test chat** on this page: dialog (multi-turn) or a single request, with a profile picker. Use it to confirm a key and model before enabling an addon that depends on LLM Access.
