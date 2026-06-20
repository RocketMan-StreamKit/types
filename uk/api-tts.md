# Озвучення тексту (`tts`)

**Потрібно:** `TTS`

Відтворює синтезоване мовлення через налаштований користувачем TTS-движок (Piper, ElevenLabs або Windows SAPI). Враховує головний перемикач TTS, голосові налаштування та гучність з UI застосунку.

## Підключення

1. Додайте `"TTS"` до `permissions` у `manifest.json` (користувач підтверджує під час встановлення).
2. Перевіряйте під час виконання через `permissions.has(AddonsPermission.TTS)`.
3. Викликайте `tts.getEngine()`, щоб дізнатися, чи увімкнено озвучення і який движок активний.

```json
{
  "id": "my-addon",
  "permissions": ["TTS", "DASHBOARD_EVENTS"]
}
```

## Як це працює

- Використовує **ту саму** конфігурацію TTS, що й вбудовані налаштування «Text to speech» (движок, голоси, перемикач, гучність).
- Звук відтворюється в **головному вікні** (той самий шлях, що у внутрішнього `speakWithTTS`).
- **Мова:** передайте `options.language` (`en`, `ru`, `uk`) або опустіть — fastText визначає uk/ru/en автоматично (fallback на англійську).
- **Гучність:** `options.volumeMultiplier` масштабує відтворення відносно гучності TTS користувача (`0` = тиша, `1` = повна гучність користувача). Значення вище `1` обмежуються до `1` — аддон не може зробити озвучення гучнішим за налаштування користувача.
- **Черга:** TTS-менеджер основного процесу серіалізує паралельні виклики `speak` (від аддонів і від застосунку).

## `tts.getEngine()`

Повертає активний движок і чи увімкнено озвучення в налаштуваннях.

```js
const { success, engine, enabled, message } = await tts.getEngine();
// success: boolean
// engine: 'piper' | 'elevenlabs' | 'windows'  (коли success)
// enabled: boolean                             (коли success)
// message: string                              (коли !success)
```

| Поле | Опис |
| --- | --- |
| `engine` | Активний движок синтезу з налаштувань користувача |
| `enabled` | `false`, якщо користувач вимкнув TTS глобально — `speak` поверне `{ success: false }` |

## `tts.speak(text, options?)`

| Параметр | Обов'язковий | Опис |
| --- | --- | --- |
| `text` | так | Текст для озвучення (обрізається; порожній текст — помилка) |
| `options.language` | ні | `en`, `ru` або `uk`. Якщо не вказано — мова визначається автоматично |
| `options.volumeMultiplier` | ні | `0`…`1` відносно гучності TTS користувача. Значення вище `1` обмежуються до `1` |

```js
await tts.speak('Дякую за фоллоу!');

await tts.speak('Дякую за донат!', { language: 'uk' });

await tts.speak('Тихе сповіщення', { volumeMultiplier: 0.5 });
```

Повертає `{ success: boolean, message?: string }`.

Типові значення `message` при `success: false`:

| Повідомлення | Причина |
| --- | --- |
| `TTS is disabled` | Користувач вимкнув TTS у налаштуваннях |
| `Text is empty` | Порожній або з пробілів `text` |
| `No TTS model configured for this language` | Движок Piper, немає моделі для визначеної мови |
| `Permission TTS is required…` | Немає дозволу в маніфесті (перевірка в пісочниці) |
| Помилки ElevenLabs / Windows | Квота API, голос не обрано, платформа недоступна тощо |

## Приклад: озвучення при події дашборду

```js
if (!permissions.has(AddonsPermission.TTS)) {
  console.warn('TTS не надано');
  return;
}

events.On('onDonation', async payload => {
  const { success, enabled } = await tts.getEngine();
  if (!success || !enabled) {
    return;
  }

  const username = payload.body?.username || 'глядач';
  await tts.speak(`Дякую ${username} за донат!`, {
    volumeMultiplier: 0.8,
  });
});
```

Див. також [Дозволи](./permissions.md) та [Огляд API](./api-overview.md).
