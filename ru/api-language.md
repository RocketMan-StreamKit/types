# Определение языка (`language`)

Определяет язык произвольного текста той же моделью fastText, что и TTS (`lid.176.ftz`, ~176 языков). **Дополнительное разрешение не требуется.**

В отличие от автоопределения TTS (которое сводит результат только к `en` / `ru` / `uk`), этот API возвращает **полный** ISO-код языка из модели (например `de`, `fr`, `ja`, `pl`, `es`).

## Как это работает

- Используется fastText [lid.176.ftz](https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.ftz) (скачивается при первом использовании в user data приложения).
- Возвращает ISO-639-1 (`language`) и, если доступно, ISO-639-3 (`alpha3`) плюс английское имя `name`.
- Если модель недоступна — запасной вариант по кириллице (`uk` при наличии і/ї/є/ґ, иначе `ru`). Другие письменности без загруженной модели не классифицируются.
- Пустой текст / только пробелы — `{ success: false }`.

## `language.detect(text)`

| Параметр | Обязателен | Описание |
| --- | --- | --- |
| `text` | да | Сообщение или произвольная строка для классификации |

```js
const res = await language.detect('Bonjour le monde!');
if (res.success) {
  console.log(res.language); // 'fr'
  console.log(res.alpha3);   // 'fra' (если есть)
  console.log(res.name);     // 'French' (если есть)
}

const ja = await language.detect('こんにちは');
// { success: true, language: 'ja', alpha3: 'jpn', name: 'Japanese' }

const empty = await language.detect('   ');
// { success: false, message: 'Text is empty' }
```

| Поле | Описание |
| --- | --- |
| `language` | Двухбуквенный ISO-639-1 (например `en`, `ru`, `uk`, `de`, `ja`) |
| `alpha3` | Трёхбуквенный ISO-639-3, если модель его вернула |
| `name` | Английское имя языка из модели, если доступно |

Частые значения `message` при `success: false`:

| Message | Причина |
| --- | --- |
| `Text is empty` | Пустой `text` или только пробелы |
| `Language could not be detected` | Модель недоступна / неуверена и нет кириллической подсказки |

## Пример: маршрутизация чата по языку

```js
events.On('onChatMessage', async payload => {
  const text = payload.body?.message || '';
  const res = await language.detect(text);
  if (!res.success) {
    return;
  }

  if (res.language === 'uk') {
    // украинский
  } else if (res.language === 'ru') {
    // русский
  } else if (res.language === 'en') {
    // английский
  } else {
    console.log('Другой язык:', res.language, res.name);
  }
});
```

## Связь с TTS

- `tts.speak` по-прежнему автоопределяет только `en` / `ru` / `uk` (fallback на английский), если `options.language` не указан.
- Используйте `language.detect`, когда нужен расширенный набор языков; при озвучке передайте явный `options.language` в `tts.speak`, если сами сопоставляете результат с поддерживаемым голосом TTS.

См. также [Озвучка текста](./api-tts.md), [Локализация](./localization.md) и [Обзор API](./api-overview.md).
