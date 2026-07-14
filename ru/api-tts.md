# Озвучка текста (`tts`)

**Требует:** `TTS`

Воспроизводит синтезированную речь через настроенный пользователем TTS-движок (Piper, ElevenLabs или Windows SAPI). Учитывает главный переключатель TTS, голосовые настройки и громкость из UI приложения.

## Подключение

1. Добавьте `"TTS"` в `permissions` в `manifest.json` (пользователь подтверждает при установке).
2. Проверяйте в рантайме через `permissions.has(AddonsPermission.TTS)`.
3. Вызывайте `tts.getEngine()`, чтобы узнать, включена ли озвучка и какой движок активен.

```json
{
  "id": "my-addon",
  "permissions": ["TTS", "DASHBOARD_EVENTS"]
}
```

## Как это работает

- Использует **ту же** конфигурацию TTS, что и встроенные настройки «Text to speech» (движок, голоса, переключатель, громкость).
- Звук воспроизводится в **главном окне** (тот же путь, что у внутреннего `speakWithTTS`).
- **Язык:** передайте `options.language` (`en`, `ru`, `uk`) или опустите — fastText определяет uk/ru/en автоматически (fallback на английский).
- Для **полного** набора языков (~176 ISO-кодов) отдельно используйте [`language.detect`](./api-language.md) (разрешение TTS не нужно). Сам TTS по-прежнему использует только `en` / `ru` / `uk`.
- **Громкость:** `options.volumeMultiplier` масштабирует воспроизведение относительно громкости TTS пользователя (`0` = тишина, `1` = полная громкость пользователя). Значения выше `1` ограничиваются до `1` — аддон не может сделать озвучку громче настроек пользователя.
- **Очередь:** TTS-менеджер основного процесса сериализует параллельные вызовы `speak` (от аддонов и от приложения).

## `tts.getEngine()`

Возвращает активный движок и включена ли озвучка в настройках.

```js
const { success, engine, enabled, message } = await tts.getEngine();
// success: boolean
// engine: 'piper' | 'elevenlabs' | 'windows'  (при success)
// enabled: boolean                             (при success)
// message: string                              (при !success)
```

| Поле | Описание |
| --- | --- |
| `engine` | Активный движок синтеза из настроек пользователя |
| `enabled` | `false`, если пользователь отключил TTS глобально — `speak` вернёт `{ success: false }` |

## `tts.speak(text, options?)`

| Параметр | Обязателен | Описание |
| --- | --- | --- |
| `text` | да | Текст для озвучки (обрезается; пустой текст — ошибка) |
| `options.language` | нет | `en`, `ru` или `uk`. Если не указан — язык определяется автоматически |
| `options.volumeMultiplier` | нет | `0`…`1` относительно громкости TTS пользователя. Значения выше `1` ограничиваются до `1` |

```js
await tts.speak('Спасибо за фоллоу!');

await tts.speak('Спасибо за донат!', { language: 'ru' });

await tts.speak('Тихое уведомление', { volumeMultiplier: 0.5 });
```

Возвращает `{ success: boolean, message?: string }`.

Типичные значения `message` при `success: false`:

| Сообщение | Причина |
| --- | --- |
| `TTS is disabled` | Пользователь отключил TTS в настройках |
| `Text is empty` | Пустой или состоящий из пробелов `text` |
| `No TTS model configured for this language` | Движок Piper, нет модели для определённого языка |
| `Permission TTS is required…` | Нет разрешения в манифесте (проверка в песочнице) |
| Ошибки ElevenLabs / Windows | Квота API, голос не выбран, платформа недоступна и т.д. |

## Пример: озвучка при событии дашборда

```js
if (!permissions.has(AddonsPermission.TTS)) {
  console.warn('TTS не выдан');
  return;
}

events.On('onDonation', async payload => {
  const { success, enabled } = await tts.getEngine();
  if (!success || !enabled) {
    return;
  }

  const username = payload.body?.username || 'зритель';
  await tts.speak(`Спасибо ${username} за донат!`, {
    volumeMultiplier: 0.8,
  });
});
```

См. также [Разрешения](./permissions.md) и [Обзор API](./api-overview.md).
