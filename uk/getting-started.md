# Початок роботи

## Що таке інтеграційний аддон?

Інтеграційний аддон — це папка, яку користувач обирає під час встановлення. Вона містить як мінімум `manifest.json`. Залежно від категорії аддона можуть також бути:

- `index.js` — код worker, що виконується у VM-пісочниці
- Статичні web-файли (`index.html`, CSS, зображення) для UI оверлею, віджета або застосунку
- Медіафайли для простого відтворення оверлею

Застосунок зберігає шлях встановлення в `Config.data.addonsInstalled[].path`. Аддони **не** входять до основного бінарного файлу застосунку.

## Архітектура

| Шар | Роль |
| --- | --- |
| **Папка аддона** | `manifest.json` + `index.js` та/або web-ресурси |
| **Main process** | Запускає worker, застосовує дозволи, маршрутизує HTTP/config IPC, обслуговує статичні маршрути |
| **Worker sandbox** | `GenerateContext()` + `vm.runInContext` — глобальні об'єкти, доступні коду аддона |

Код аддона має використовувати **глобальні** імена з контексту пісочниці. `import` SDK StreamKit+ всередині VM не підтримується.

## Встановлення аддона

1. Зберіть або підготуйте папку з `manifest.json` (і необхідними файлами).
2. Відкрийте **Settings** → розділ, що відповідає `type` вашого аддона (Platforms, Overlay, Widgets, Applications).
3. **Install from folder** і оберіть каталог.
4. Підтвердіть запитані **permissions** під час встановлення.
5. Увімкніть аддон і налаштуйте параметри, якщо worker викликає `GenerateConfig()`.

Для worker-аддонів на TypeScript: скомпілюйте в `index.js` перед встановленням або встановіть папку з виходом webpack.

## Швидкий старт проєкту

Використовуйте **`@rocketman-streamkit/addon-generator`**, щоб розгорнути новий репозиторій аддона без ручного копіювання файлів.

```bash
npx @rocketman-streamkit/addon-generator
```

Аліас: `npx create-streamkit-addon`.

**Навіщо:** інтерактивний майстер готує проєкт під обрану категорію аддона — `manifest.json` з потрібним `type` і дозволами, заготовки worker або web-файлів, TypeScript-конфіг з актуальною версією [`@rocketman-streamkit/types`](https://www.npmjs.com/package/@rocketman-streamkit/types) з npm, а також GitHub Actions для релізів (`main.zip`, `manifest.json`, іконка) з перевіркою, що тег релізу ще не існує.

Майстер запитає каталог виводу, категорію аддона (платформа, оверлей, віджет, застосунок, гра тощо), варіант з worker або статичний, JavaScript чи TypeScript, поля маніфесту (`id`, локалізована назва/опис, `app_version`, …) та опційний workflow синхронізації з каталогом.

Потрібен Node.js 18+. Запускайте в порожній папці або вкажіть підкаталог, який буде створено.

## Мінімальний приклад worker

```js
await network.endpoints.create('hook', 'POST', 'onHook');

events.On('onHook', ({ body }) => {
  console.log('Webhook payload', body);
  return { ok: true };
});
```

URL ендпоінта:

```
http://localhost:{WEB_SERVER_PORT}/addon/{addonId}/hook
```

Замініть `{addonId}` на `id` з `manifest.json`, а `{WEB_SERVER_PORT}` — на локальний порт web-сервера застосунку.

## Структура проєкту (рекомендовано)

```
MyOrg-my-stream-addon/
├── manifest.json       # id: "MyOrg/my-stream-addon"
├── index.js            # або index.ts + tsconfig.json (збірка в index.js)
├── logo.png            # іконка з маніфесту
├── index.html          # якщо потрібен web UI
└── ...                 # додаткові ресурси з web_contents
```

## Пов'язані матеріали

- [manifest.json](./manifest.md) — обов'язкові поля, зокрема `id`
- [Публікація та релізи](./publishing.md) — реєстрація в каталозі, файли GitHub-релізу (`main.zip`, `manifest.json`, іконка)
- [Налаштування TypeScript](./typescript.md) — типи з `@rocketman-streamkit/types`
- [Дозволи](./permissions.md) — оголошуйте лише те, що потрібно
