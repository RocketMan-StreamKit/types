# Localization

Integration addons run in an isolated worker and **must not** use the StreamKit+ app language system (`src/shared/lang/`). Do not use translation keys, import app language files, or copy strings from them.

## App UI locale (`LANG`)

The sandbox exposes a read-only bridge to the user's **UI locale** in settings:

| Member | Description |
| --- | --- |
| `LANG.current` | `'en'`, `'ru'`, or `'uk'` — matches the app language setting |
| `LANG.onChangeLanguage(cb)` | Called when the user changes language; returns `{ Destroy }` to unsubscribe |

No permission is required. Use `LANG.current` to pick the right string from your own per-locale objects (see below). This does **not** give access to app translation keys or JSON files.

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
// later: sub.Destroy();
```

## Per-locale objects

Provide user-visible text as objects with locale keys:

```js
{
  en: 'Connected',
  ru: 'Подключено',
  uk: 'Підключено',
}
```

`en` is required where the API expects `DashboardLocalizedText`. The UI falls back to `en` when the user's locale is missing.

## Where this applies

| API | Localized fields |
| --- | --- |
| `manifest.name` / `description` | `en`, `ru`, `uk` |
| `GenerateConfig` editor `label`, `description`, `placeholder` | `en`, `ru`, `uk` |
| `status.Update({ message })` | `en`, `ru`, `uk` |
| `notify.Send({ title, message })` | per-locale or plain string |
| `dashboard.registerPlatform({ name })` | per-locale |
| `dashboard.addRecord({ message })` | string, per-locale object, or app `LangData` tuple* |
| `dashboard.addChatMessage({ content })` | same as `message` |
| `dashboard.registerTriggers` labels | per-locale |

\* `LangData` tuples in dashboard messages are resolved by the **app** UI, not by addon translation files. Prefer explicit `{ en, ru?, uk? }` objects in third-party addons for self-contained localization.

## Plain strings

Many APIs also accept a plain `string` when you do not need multiple locales.
