# Налаштування TypeScript

Worker-аддони можна писати на TypeScript. Типи беруться з npm-пакета **`@rocketman-streamkit/types`** — не додавайте `@types/node` або інші пакети типів з `node_modules` до проєкту аддона.

## Встановлення

```bash
npm install --save-dev @rocketman-streamkit/types
```

Встановлюйте версію пакета, що збігається з цільовою версією StreamKit+ (той самий `major.minor.patch`, що й `app_version` у `manifest.json`, якщо ви залежите від нових API пісочниці).

## tsconfig.json

Розмістіть `tsconfig.json` поруч із `index.ts` у папці аддона:

```json
{
  "compilerOptions": {
    "types": [],
    "lib": ["ES2020"],
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["./**/*.ts", "./node_modules/@rocketman-streamkit/types/addon.d.ts"]
}
```

Або підключіть поле `types` пакета напряму:

```json
{
  "compilerOptions": {
    "types": ["@rocketman-streamkit/types"],
    "lib": ["ES2020"],
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["./**/*.ts"]
}
```

Не додавайте `@types/node` — worker-аддони не є процесами Node.js.

Статичні аддони (лише HTML/JS, без `index.ts`) не потребують `tsconfig.json`.

## Глобальні об'єкти

`addon.d.ts` з `@rocketman-streamkit/types` оголошує глобальні об'єкти пісочниці:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- enum `AddonsPermission`
- `GenerateConfig`, `data`, `isDeveloperMode`

Імпорти в TypeScript аддона не потрібні — використовуйте глобальні об'єкти напряму.

## Оновлення типів

Коли в новій версії StreamKit+ з'являються потрібні вам API пісочниці, оновіть dev-залежність до відповідної версії пакета:

```bash
npm install --save-dev @rocketman-streamkit/types@1.0.0
```

Замініть `1.0.0` на версію StreamKit+, яка надає потрібні API.

## Вихід збірки

Скомпілюйте або зберіть `index.ts` у `index.js` у папці встановлення. Worker виконує лише `index.js`. Тримайте `manifest.json` і ресурси поруч із зібраним файлом.

Статичні аддони без worker можна встановлювати напряму з вихідників (`manifest.json`, іконка, HTML, `web_contents`).
