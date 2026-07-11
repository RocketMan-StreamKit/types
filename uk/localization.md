# Локалізація

Інтеграційні аддони працюють у ізольованому worker і **не повинні** використовувати мовну систему застосунку StreamKit+ (`src/shared/lang/`). Не використовуйте ключі перекладів, не імпортуйте мовні файли застосунку і не копіюйте рядки з них.

## Локаль UI застосунку (`LANG`)

Пісочниця надає доступ лише для читання до **локалі UI** користувача в налаштуваннях:

| Член | Опис |
| --- | --- |
| `LANG.current` | `'en'`, `'ru'` або `'uk'` — відповідає мові застосунку |
| `LANG.onChangeLanguage(cb)` | Викликається при зміні мови; повертає `{ Destroy }` для відписки |

Дозвіл не потрібен. Використовуйте `LANG.current`, щоб обрати потрібний рядок з власних об'єктів за локалями (див. нижче). Це **не** дає доступу до ключів перекладів або JSON-файлів застосунку.

Щоб визначити мову **тексту повідомлення** (не локаль UI), використовуйте [`language.detect`](./api-language.md) — повертає ISO-коди з fastText lid.176 (~176 мов) і не залежить від `LANG`.

```js
const labels = {
  en: 'Connected',
  ru: 'Подключено',
  uk: 'Підключено',
};

status.Update({ current: 'online', message: { [LANG.current]: labels[LANG.current] } });

const sub = LANG.onChangeLanguage((lang) => {
  console.log('UI locale:', lang);
});
// пізніше: sub.Destroy();
```

## Об'єкти за локалями

Надавайте текст, видимий користувачу, як об'єкти з ключами локалей:

```js
{
  en: 'Connected',
  ru: 'Подключено',
  uk: 'Підключено',
}
```

`en` обов'язковий там, де API очікує `DashboardLocalizedText`. UI використовує `en` як запасний варіант, якщо локаль користувача відсутня.

## Де це застосовується

| API | Локалізовані поля |
| --- | --- |
| `manifest.name` / `description` | `en`, `ru`, `uk` |
| `GenerateConfig` — `label`, `description`, `placeholder` в `editor` | `en`, `ru`, `uk` |
| `status.Update({ message })` | `en`, `ru`, `uk` |
| `notify.Send({ title, message })` | за локалями або звичайний рядок |
| `dashboard.registerPlatform({ name })` | за локалями |
| `dashboard.addRecord({ message })` | рядок, об'єкт за локалями або кортеж app `LangData`* |
| `dashboard.addChatMessage({ content })` | те саме, що для `message` |
| мітки `dashboard.registerTriggers` | за локалями |

\* Кортежі `LangData` у повідомленнях dashboard розв'язує UI **застосунку**, а не файли перекладів аддона. Для самодостатньої локалізації в сторонніх аддонах краще використовувати явні об'єкти `{ en, ru?, uk? }`.

## Звичайні рядки

Багато API також приймають звичайний `string`, коли кілька локалей не потрібні.
