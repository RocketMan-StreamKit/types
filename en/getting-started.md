# Getting started

## What is an integration addon?

An integration addon is a folder the user selects at install time. It contains at minimum `manifest.json`. Depending on the addon category it may also include:

- `index.js` — worker code executed in the sandbox VM
- Static web files (`index.html`, CSS, images) for overlay, widget, or application UIs
- Media files for simple overlay playback

The app stores the install path in `Config.data.addonsInstalled[].path`. Addons are **not** bundled inside the main application binary.

## Architecture

| Layer | Role |
| --- | --- |
| **Addon folder** | `manifest.json` + `index.js` and/or web assets |
| **Main process** | Spawns worker, enforces permissions, bridges HTTP/config IPC, serves static routes |
| **Worker sandbox** | `GenerateContext()` + `vm.runInContext` — globals exposed to addon code |

Addon code must use **global** names from the sandbox context. `import` of the StreamKit+ SDK inside the VM is not supported.

## Installing an addon

1. Build or prepare a folder with `manifest.json` (and required files).
2. Open **Settings** → the section matching your addon `type` (Platforms, Overlay, Widgets, Applications).
3. **Install from folder** and select the directory.
4. Approve requested **permissions** at install time.
5. Enable the addon and configure settings if the worker calls `GenerateConfig()`.

For TypeScript worker addons: compile to `index.js` before install, or install the webpack output folder.

## Scaffold a new project

Use **`@rocketman-streamkit/addon-generator`** to bootstrap a new addon repository instead of copying files manually.

```bash
npx @rocketman-streamkit/addon-generator
```

Alias: `npx create-streamkit-addon`.

**Why use it:** the interactive wizard sets up a ready-to-edit project for your addon category — `manifest.json` with the right `type` and permissions, starter worker or web files, TypeScript typings pinned to the latest [`@rocketman-streamkit/types`](https://www.npmjs.com/package/@rocketman-streamkit/types), and GitHub Actions to build releases (`main.zip`, `manifest.json`, icon) with a check that the release tag does not already exist.

The wizard asks for the output folder, addon category (platform, overlay, widget, application, game, …), worker vs static variant, JavaScript or TypeScript, manifest fields (`id`, localized name/description, `app_version`, …), and optional catalog sync workflow.

Requires Node.js 18+. Run it in an empty directory or let it create a subfolder you specify.

## Minimal worker example

```js
await network.endpoints.create('hook', 'POST', 'onHook');

events.On('onHook', ({ body }) => {
  console.log('Webhook payload', body);
  return { ok: true };
});
```

Endpoint URL:

```
http://localhost:{WEB_SERVER_PORT}/addon/{addonId}/hook
```

Replace `{addonId}` with the `id` from `manifest.json` and `{WEB_SERVER_PORT}` with the app's local web server port.

## Project layout (recommended)

```
MyOrg-my-stream-addon/
├── manifest.json       # id: "MyOrg/my-stream-addon"
├── index.js            # or index.ts + tsconfig.json (build to index.js)
├── logo.png            # icon referenced by manifest
├── index.html          # if web UI is required
└── ...                 # additional assets listed in web_contents
```

## Related reading

- [manifest.json](./manifest.md) — required fields, especially `id`
- [Publishing and releases](./publishing.md) — catalog registration, GitHub release assets (`main.zip`, `manifest.json`, icon)
- [TypeScript setup](./typescript.md) — typings from `@rocketman-streamkit/types`
- [Permissions](./permissions.md) — declare only what you need
