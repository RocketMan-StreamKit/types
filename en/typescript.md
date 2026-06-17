# TypeScript setup

Worker addons can be authored in TypeScript. Typings come from the npm package **`@rocketman-streamkit/types`** — do not add `@types/node` or other `node_modules` type packages to the addon project.

## Installation

```bash
npm install --save-dev @rocketman-streamkit/types
```

Install the package version that matches the StreamKit+ release your addon targets (same `major.minor.patch` as `app_version` in `manifest.json` when you depend on new sandbox APIs).

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
  "include": ["./**/*.ts", "./node_modules/@rocketman-streamkit/types/addon.d.ts"]
}
```

Alternatively, reference the package types field directly:

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

Do **not** add `@types/node` — addon workers are not Node.js processes.

Static addons (HTML/JS only, no `index.ts`) do not need `tsconfig.json`.

## Globals

`addon.d.ts` from `@rocketman-streamkit/types` declares sandbox globals:

- `network`, `events`, `permissions`, `api`, `dashboard`, `addons`, `storage`, …
- `AddonsPermission` enum
- `GenerateConfig`, `data`, `isDeveloperMode`

No imports are required in addon TypeScript — use globals directly.

## Updating typings

When StreamKit+ ships a new sandbox API in a release you target, bump the dev dependency to the matching package version:

```bash
npm install --save-dev @rocketman-streamkit/types@1.0.0
```

Replace `1.0.0` with the StreamKit+ version that provides the APIs you need.

## Build output

Compile or bundle `index.ts` to `index.js` in the install folder. The worker executes `index.js` only. Keep `manifest.json` and assets alongside the built file.

Static addons without a worker can be installed directly from source (`manifest.json`, icon, HTML, `web_contents`).
