# Ambient types for `@tamer4lynx` packages

Native Tamer packages that add **custom elements** (JSX intrinsics) or **ambient** APIs should expose typings in a way the **t4l** CLI can discover and aggregate into **`.tamer/tamer-components.d.ts`** at the app root.

## `package.json` contract

1. **`exports`** — Declare a subpath whose `types` points at the emitted `.d.ts` (for example `./webview-jsx` → `./dist/webview-jsx.d.ts`).
2. **`tamer.ambientTypeExports`** — Optional string array of those subpaths (same strings as in `exports`, e.g. `["./webview-jsx"]`). If omitted, **t4l** falls back to conventional export keys such as `./webview-jsx` and `./icon-jsx`.

## Type modules

Implement ambient tags in **`src/*-jsx.ts`** (not only `.d.ts`): `declare module '@lynx-js/types' { interface IntrinsicElements { … } }` merged into the existing interface, plus **`/// <reference types="@lynx-js/types" />`**. **`.ts`** is required so **`import './icon-jsx'`** resolves when apps bundle source (e.g. Rspeedy aliases to `packages/tamer-icons/src`). **`tsc`** emits **`dist/*-jsx.js`** (usually `export {}`) and **`dist/*-jsx.d.ts`**; the post-build script adjusts **`dist/index.d.ts`** to **`/// <reference path="./…-jsx.d.ts" />`** for consumers. **t4l**’s `.tamer/tamer-components.d.ts` uses **triple-slash `path`** references into **`node_modules/.../dist/*-jsx.d.ts`** ( **`types=`** subpaths are unreliable for augmentations in some setups).

## CLI behavior

When **`syncTamerComponentTypes`** in `tamer.config.json` is not `false` (default: on), **`t4l init`** and **`t4l link`** regenerate `.tamer/tamer-components.d.ts` and ensure **`tsconfig.json`** includes it (and remove broad `node_modules/@tamer4lynx/tamer-*/**/*.d.ts` globs).
