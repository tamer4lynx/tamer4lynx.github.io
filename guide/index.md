# Guide

## What you are looking at

**Lynx** is the cross-platform rendering engine and JS runtime ([docs](https://lynxjs.org/)). You write a Lynx app (often with **ReactLynx** and **Rspeedy**).

**Tamer4Lynx** is the layer *around* that app for real products: a **CLI** (`t4l` from `@tamer4lynx/cli`) plus **npm packages** (`@tamer4lynx/*`) that add routing, native shell UI, networking helpers, auth, secure storage, deep links, and more — with **iOS and Android host projects** generated or linked by the CLI. **HarmonyOS** support is coming soon.

If you only remember three things:

1. **Create or open a Lynx project**, then run **`t4l init`** where `tamer.config.json` should live.
2. **Add Tamer packages with the CLI** (`t4l add`, `t4l add-core`, or `t4l add-dev`), not raw `npm install …@latest`, so versions match the **highest published semver** on the registry; then **`t4l link`**.
3. Use **[Getting Started](/guide/getting-started)** for the full flow (dev server, debug builds, when to run `t4l signing`).

## Where to read next

| Topic | Page |
|-------|------|
| End-to-end setup | [Getting Started](/guide/getting-started) — CLI, `t4l init`, `t4l start`, builds |
| `tamer.config.json`, autolink, extensions | [Configuration](/guide/configuration) |
| How the monorepo example wires packages | [Example Anatomy](/guide/example-anatomy) |
| Every `t4l` subcommand | [Commands](/reference/commands) |
| Package list by category | [Packages](/packages/) → [Core](/packages/core/), [UI](/packages/ui/), [Platform](/packages/platform/), [Tooling](/packages/tooling/) |

## Lynx ecosystem

[Lynx](https://lynxjs.org/) · [Rspeedy](https://lynxjs.org/rspeedy/) · [ReactLynx](https://lynxjs.org/react/)
