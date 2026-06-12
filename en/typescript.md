# TypeScript setup

Worker addons can be authored in TypeScript. Typings come **only** from `data/addon.d.ts` — do not add `@types/node` or other `node_modules` type packages to the addon project.

## tsconfig.json

Place `tsconfig.json` next to `index.ts` in your addon folder:

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

Adjust the path to `addon.d.ts` relative to your addon folder (the example assumes the addon lives two levels below the repo root).

Static addons (HTML/JS only, no `index.ts`) do not need `tsconfig.json`.

## Globals

`data/addon.d.ts` declares sandbox globals:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- `AddonsPermission` enum
- `GenerateConfig`, `data`, `isDeveloperMode`

No imports are required in addon TypeScript — use globals directly.

## Regenerating typings

When the sandbox API changes in a new StreamKit+ version:

```bash
npm run type:addons
```

This updates `data/addon.d.ts` and `dist/data/addon.d.ts` from JSDoc in the integrations worker source.

## Build output

Compile or bundle `index.ts` to `index.js` in the install folder. The worker executes `index.js` only. Keep `manifest.json` and assets alongside the built file.

Static addons without a worker can be installed directly from source (`manifest.json`, icon, HTML, `web_contents`).
