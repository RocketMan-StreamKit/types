# Определение языка (`language`)

Определяет язык произвольного текста той же моделью fastText, что и TTS (`lid.176.ftz`, ~176 языков). **Дополнительное разрешение не требуется.**

В отличие от автоопределения TTS (которое сводит результат только к `en` / `ru` / `uk`), этот API возвращает **полный** ISO-код языка из модели (например `de`, `fr`, `ja`, `pl`, `es`).

## Как это работает

- Используется fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (скачивается при первом использовании в user data приложения).
- **Уверенно:** вероятность топ-метки ≥ `0.4` и отрыв от 2-го места ≥ `0.25` → `{ success: true, language, alpha3?, name? }`.
- **Неуверенно:** иначе → `{ success: false, message, possibles }` с ранжированными кандидатами (без одного `language`). Решение о переводе — за аддоном.
- Пустой текст / только пробелы → `{ success: false, message: 'Text is empty' }` (без `possibles`).
- Если модель недоступна → `{ success: false, message, possibles: [] }`.

## `language.detect(text)`

| Параметр | Обязателен | Описание |
| --- | --- | --- |
| `text` | да | Сообщение или произвольная строка для классификации |

```js
const res = await language.detect('Bonjour le monde!');
if (res.success) {
  console.log(res.language); // 'fr'
  console.log(res.alpha3);   // 'fra' (когда есть)
  console.log(res.name);     // 'French' (когда есть)
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

### Поля при успехе

| Поле | Описание |
| --- | --- |
| `language` | ISO-639-1 двухбуквенный код (например `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | ISO-639-3 трёхбуквенный код, если модель его даёт |
| `name` | Английское имя языка, если доступно |

### Поля при ошибке

| Поле | Описание |
| --- | --- |
| `message` | Человекочитаемая причина |
| `possibles` | Опциональный ранжированный список `{ language, probability }[]` (сначала наиболее вероятные, до 5). Есть, когда модель отработала, но не уверена; может быть `[]` при сбое модели; отсутствует для пустого текста |

Типичные значения `message` при `success: false`:

| Сообщение | Причина |
| --- | --- |
| `Text is empty` | Пустой `text` или только пробелы |
| `Language is difficult to determine` | Низкая уверенность, близкие топ-языки или модель недоступна |

## Пример: переводить только при уверенном результате

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    // Опционально: разобрать res.possibles и решить вручную
    // например if (res.possibles?.[0]?.probability > 0.3) …
    return;
  }

  if (res.language === 'uk' || res.language === 'ru') {
    // перевести…
  }
});
```

## Связь с TTS

- `tts.speak` по-прежнему автоопределяет только `en` / `ru` / `uk` (запасной английский / кириллическая подсказка), если `options.language` не задан.
- Используйте `language.detect`, когда нужен расширенный набор языков; при озвучке передайте явный `options.language` в `tts.speak`, если сами сопоставляете результат с поддерживаемым голосом TTS.

См. также [Text-to-speech](./api-tts.md), [Localization](./localization.md) и [API overview](./api-overview.md).
