# Налаштування TypeScript

Worker-аддони можна писати на TypeScript. Типи беруться **лише** з `data/addon.d.ts` — не додавайте `@types/node` або інші пакети типів з `node_modules` до проєкту аддона.

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
  "include": ["./**/*.ts", "../../data/addon.d.ts"]
}
```

Налаштуйте шлях до `addon.d.ts` відносно папки аддона (приклад припускає, що аддон знаходиться на два рівні нижче кореня репозиторію).

Статичні аддони (лише HTML/JS, без `index.ts`) не потребують `tsconfig.json`.

## Глобальні об'єкти

`data/addon.d.ts` оголошує глобальні об'єкти пісочниці:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- enum `AddonsPermission`
- `GenerateConfig`, `data`, `isDeveloperMode`

Імпорти в TypeScript аддона не потрібні — використовуйте глобальні об'єкти напряму.

## Регенерація типів

Коли API пісочниці змінюється в новій версії StreamKit+:

```bash
npm run type:addons
```

Це оновлює `data/addon.d.ts` і `dist/data/addon.d.ts` з JSDoc у вихідному коді integrations worker.

## Вихід збірки

Скомпілюйте або зберіть `index.ts` у `index.js` у папці встановлення. Worker виконує лише `index.js`. Тримайте `manifest.json` і ресурси поруч із зібраним файлом.

Статичні аддони без worker можна встановлювати напряму з вихідників (`manifest.json`, іконка, HTML, `web_contents`).
