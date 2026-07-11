# Визначення мови (`language`)

Визначає мову довільного тексту тією ж моделлю fastText, що й TTS (`lid.176.ftz`, ~176 мов). **Додатковий дозвіл не потрібен.**

На відміну від автовизначення TTS (яке зводить результат лише до `en` / `ru` / `uk`), цей API повертає **повний** ISO-код мови з моделі (наприклад `de`, `fr`, `ja`, `pl`, `es`).

## Як це працює

- Використовується fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (завантажується при першому використанні в user data застосунку).
- Повертає ISO-639-1 (`language`) і, якщо доступно, ISO-639-3 (`alpha3`) плюс англійську назву `name`.
- Якщо модель недоступна — запасний варіант за кирилицею (`uk` за наявності і/ї/є/ґ, інакше `ru`). Інші письменності без завантаженої моделі не класифікуються.
- Порожній текст / лише пробіли — `{ success: false }`.

## `language.detect(text)`

| Параметр | Обовʼязковий | Опис |
| --- | --- | --- |
| `text` | так | Повідомлення або довільний рядок для класифікації |

```js
const res = await language.detect('Bonjour le monde!');
if (res.success) {
  console.log(res.language); // 'fr'
  console.log(res.alpha3);   // 'fra' (якщо є)
  console.log(res.name);     // 'French' (якщо є)
}

const ja = await language.detect('こんにちは');
// { success: true, language: 'ja', alpha3: 'jpn', name: 'Japanese' }

const empty = await language.detect('   ');
// { success: false, message: 'Text is empty' }
```

| Поле | Опис |
| --- | --- |
| `language` | Дволітерний ISO-639-1 (наприклад `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | Трилітерний ISO-639-3, якщо модель його повернула |
| `name` | Англійська назва мови з моделі, якщо доступна |

Поширені значення `message` при `success: false`:

| Message | Причина |
| --- | --- |
| `Text is empty` | Порожній `text` або лише пробіли |
| `Language could not be detected` | Модель недоступна / невпевнена і немає кириличної підказки |

## Приклад: маршрутизація чату за мовою

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    return;
  }

  if (res.language === 'uk') {
    // українська
  } else if (res.language === 'ru') {
    // російська
  } else if (res.language === 'en') {
    // англійська
  } else {
    console.log('Інша мова:', res.language, res.name);
  }
});
```

## Звʼязок із TTS

- `tts.speak` як і раніше автовизначає лише `en` / `ru` / `uk` (fallback на англійську), якщо `options.language` не вказано.
- Використовуйте `language.detect`, коли потрібен розширений набір мов; під час озвучення передайте явний `options.language` у `tts.speak`, якщо самі зіставляєте результат із підтримуваним голосом TTS.

Див. також [Озвучення тексту](./api-tts.md), [Локалізація](./localization.md) та [Огляд API](./api-overview.md).
