# Визначення мови (`language`)

Визначає мову довільного тексту тією ж моделлю fastText, що й TTS (`lid.176.ftz`, ~176 мов). **Додатковий дозвіл не потрібен.**

На відміну від автовизначення TTS (яке зводить результат лише до `en` / `ru` / `uk`), цей API повертає **повний** ISO-код мови з моделі (наприклад `de`, `fr`, `ja`, `pl`, `es`).

## Як це працює

- Використовується fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (завантажується при першому використанні в user data застосунку).
- **Впевнено:** ймовірність топ-мітки ≥ `0.4` і відрив від 2-го місця ≥ `0.25` → `{ success: true, language, alpha3?, name? }`.
- **Невпевнено:** інакше → `{ success: false, message, possibles }` з ранжованими кандидатами (без одного `language`). Рішення про переклад — за аддоном.
- Порожній текст / лише пробіли → `{ success: false, message: 'Text is empty' }` (без `possibles`).
- Якщо модель недоступна → `{ success: false, message, possibles: [] }`.

## `language.detect(text)`

| Параметр | Обовʼязковий | Опис |
| --- | --- | --- |
| `text` | так | Повідомлення або довільний рядок для класифікації |

```js
const res = await language.detect('Bonjour le monde!');
if (res.success) {
  console.log(res.language); // 'fr'
  console.log(res.alpha3);   // 'fra' (коли є)
  console.log(res.name);     // 'French' (коли є)
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

### Поля при успіху

| Поле | Опис |
| --- | --- |
| `language` | ISO-639-1 дволітерний код (наприклад `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | ISO-639-3 трилітерний код, якщо модель його дає |
| `name` | Англійська назва мови, якщо доступна |

### Поля при помилці

| Поле | Опис |
| --- | --- |
| `message` | Людинозрозуміла причина |
| `possibles` | Опційний ранжований список `{ language, probability }[]` (спочатку найімовірніші, до 5). Є, коли модель відпрацювала, але не впевнена; може бути `[]` при збої моделі; відсутній для порожнього тексту |

Типові значення `message` при `success: false`:

| Повідомлення | Причина |
| --- | --- |
| `Text is empty` | Порожній `text` або лише пробіли |
| `Language is difficult to determine` | Низька впевненість, близькі топ-мови або модель недоступна |

## Приклад: перекладати лише при впевненому результаті

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    // Опційно: розібрати res.possibles і вирішити вручну
    // наприклад if (res.possibles?.[0]?.probability > 0.3) …
    return;
  }

  if (res.language === 'uk' || res.language === 'ru') {
    // перекласти…
  }
});
```

## Звʼязок із TTS

- `tts.speak` як і раніше автовизначає лише `en` / `ru` / `uk` (запасна англійська / кирилична підказка), якщо `options.language` не задано.
- Використовуйте `language.detect`, коли потрібен розширений набір мов; під час озвучення передайте явний `options.language` у `tts.speak`, якщо самі зіставляєте результат із підтримуваним голосом TTS.

Див. також [Text-to-speech](./api-tts.md), [Localization](./localization.md) і [API overview](./api-overview.md).
