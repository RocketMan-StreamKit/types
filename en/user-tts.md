# Text to speech

**Settings → Text to speech** reads messages aloud. Addons with permission can send text through the engine you configure here (for example donation alerts). You pick the engine, voices, and volume; API keys and local models stay in StreamKit+.

## Common controls

| Setting | What it does |
| --- | --- |
| **Enable TTS** | Master switch |
| **Speech engine** | Piper (local), ElevenLabs (cloud), or Windows TTS |
| **Volume** | 0–200% (values above 100% are amplified in the main window) |
| **Language, test phrase, Test voice** | Check the current voice before going live |

If the addon does not specify a language, StreamKit+ detects English, Russian, or Ukrainian automatically (with a fallback to English).

## Piper (default)

Local [Piper](https://github.com/rhasspy/piper) voices. No account required.

- Download and manage ONNX voice models per language (English, Russian, Ukrainian).
- Optional **CUDA** acceleration on NVIDIA GPUs (Windows and Linux).
- **Launch mode**: start the worker immediately, or on first speak (saves RAM; first line may be slower).
- Speed and expressiveness sliders.
- A status indicator shows whether the worker is running.

If the Piper worker is missing from the install, the section explains that it could not start.

## ElevenLabs

Cloud voices via the [ElevenLabs](https://elevenlabs.io/) API (`eleven_multilingual_v2`).

- Add one or more API tokens with character quotas. StreamKit+ picks the token with the most remaining quota (cached for a few minutes).
- Shared voice settings: stability, similarity, style, speed.
- Refresh limits from the panel when you need an up-to-date quota.

Needs a network connection and a valid token.

## Windows TTS

System voices (SAPI). **Windows only** — the panel is disabled on Linux and macOS.

Configure a voice, rate, and volume per language.
