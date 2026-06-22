# Настройка TypeScript

Воркеры аддонов можно писать на TypeScript. Типы берутся из npm-пакета **`@rocketman-streamkit/types`** — не добавляйте `@types/node` и другие пакеты типов из `node_modules` в проект аддона.

## Установка

```bash
npm install --save-dev @rocketman-streamkit/types
```

Устанавливайте версию пакета, совпадающую с целевой версией StreamKit+ (тот же `major.minor.patch`, что и `app_version` в `manifest.json`, если вы зависите от новых API песочницы).

## tsconfig.json

Разместите `tsconfig.json` рядом с `index.ts` в папке аддона:

```json
{
  "compilerOptions": {
    "types": ["@rocketman-streamkit/types/addon.d.ts"],
    "lib": ["ES2020"],
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true,
    "target": "ES2020"
  },
  "include": ["./**/*.ts"],
  "exclude": ["node_modules"]
}
```

Не добавляйте `@types/node` — воркеры аддонов не являются процессами Node.js.

Статические аддоны (только HTML/JS, без `index.ts`) не требуют `tsconfig.json`.

## Глобальные объекты

`addon.d.ts` из `@rocketman-streamkit/types` объявляет глобальные объекты песочницы:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- перечисление `AddonsPermission`
- `GenerateConfig`, `data`, `isDeveloperMode`

Импорты в TypeScript аддона не нужны — используйте глобальные объекты напрямую.

## Обновление типов

Когда в новой версии StreamKit+ появляются нужные вам API песочницы, обновите dev-зависимость до соответствующей версии пакета:

```bash
npm install --save-dev @rocketman-streamkit/types@1.0.0
```

Замените `1.0.0` на версию StreamKit+, которая предоставляет нужные API.

## Результат сборки

Скомпилируйте или соберите `index.ts` в `index.js` в папке установки. Воркер выполняет только `index.js`. Держите `manifest.json` и ресурсы рядом со собранным файлом.

Статические аддоны без воркера можно устанавливать напрямую из исходников (`manifest.json`, иконка, HTML, `web_contents`).
