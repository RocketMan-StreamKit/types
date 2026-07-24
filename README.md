# StreamKit+ — Addon Developer Resources

Official materials for building **StreamKit+ integration addons**: API reference, manifest guide, permissions, and TypeScript typings.

Each release is published as a **version branch** in this repository (branch name = StreamKit+ app version). The default branch **`main`** always mirrors the latest release tip.

---

## Documentation

Full addon developer documentation (English, Русский, Українська):

**[Open documentation (web) →](https://rocketman-streamkit.github.io/types/)**

Browse markdown in this repository from [`index.md`](./index.md) or [`index.html`](./index.html) on `main` (latest) or on a version branch for a specific StreamKit+ release.

---

## TypeScript typings

Install sandbox API declarations from npm — global typings for `network`, `events`, `dashboard`, `GenerateConfig`, and the rest of the VM API:

**[@rocketman-streamkit/types on npm](https://www.npmjs.com/package/@rocketman-streamkit/types)**

```bash
npm install --save-dev @rocketman-streamkit/types
```

Install the package version that matches the StreamKit+ release you target.

---

## Repository layout

| Path | Description |
| --- | --- |
| [`index.md`](./index.md) / [`index.html`](./index.html) | Language picker — entry point for all docs |
| [`en/`](./en/) | English documentation |
| [`ru/`](./ru/) | Russian documentation |
| [`uk/`](./uk/) | Ukrainian documentation |

---

## Versioning

Documentation and typings for a given StreamKit+ release share the same semver (for example branch `1.0.0` ↔ npm `@rocketman-streamkit/types@1.0.0`). Branch `main` always points at the newest published docs tip.

When upgrading StreamKit+, switch to the matching version branch in this repo and npm package version in your addon project (or stay on `main` / `latest` for current).
