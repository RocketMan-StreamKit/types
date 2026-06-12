# Настройка TypeScript

Воркеры аддонов можно писать на TypeScript. Типы берутся **только** из `data/addon.d.ts` — не добавляйте `@types/node` и другие пакеты типов из `node_modules` в проект аддона.

## tsconfig.json

Разместите `tsconfig.json` рядом с `index.ts` в папке аддона:

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

Подстройте путь к `addon.d.ts` относительно папки аддона (в примере аддон лежит на два уровня ниже корня репозитория).

Статические аддоны (только HTML/JS, без `index.ts`) не требуют `tsconfig.json`.

## Глобальные объекты

`data/addon.d.ts` объявляет глобальные объекты песочницы:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- перечисление `AddonsPermission`
- `GenerateConfig`, `data`, `isDeveloperMode`

Импорты в TypeScript аддона не нужны — используйте глобальные объекты напрямую.

## Регенерация типов

Когда API песочницы меняется в новой версии StreamKit+:

```bash
npm run type:addons
```

Это обновляет `data/addon.d.ts` и `dist/data/addon.d.ts` из JSDoc в исходниках воркера интеграций.

## Результат сборки

Скомпилируйте или соберите `index.ts` в `index.js` в папке установки. Воркер выполняет только `index.js`. Держите `manifest.json` и ресурсы рядом со собранным файлом.

Статические аддоны без воркера можно устанавливать напрямую из исходников (`manifest.json`, иконка, HTML, `web_contents`).
