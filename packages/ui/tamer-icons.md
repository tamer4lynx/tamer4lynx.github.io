# tamer-icons

Typings and font assets for the native Lynx **`<icon>`** element (Material Icons, Font Awesome). There is **no `Icon` React component**—you use **`<icon>`** with the native attribute names below.

## Overview

- **`<icon>`** — Native custom element; **`IconElementProps`** in **`packages/tamer-icons/src/icon-jsx.ts`** extends **`ViewProps`** from **`@lynx-js/types`**, so you also get standard view props (**`className`**, **`id`**, **`style`**, …) alongside **`icon`**, **`set`**, **`size`**, **`iconColor`**.
- **`IconSet`** — `'material' | 'fontawesome' | 'fa'` (exported for app code)
- **Fonts** — Material Icons and Font Awesome codepoints. Fonts must be loaded via `lynx.addFont` or `@font-face`; the package provides **`MATERIAL_ICONS_URL`** and **`FONTAWESOME_SOLID_URL`**.

### iOS (CocoaPods)

The pod bundles **`.ttf`** files under `tamericons/Resources/`. They are produced by the package **`npm run build`** (`fetch-fonts` → **`copy-ios-fonts`**). After `npm install` / `pnpm install`, `prepare` runs that build; if you use **`--ignore-scripts`**, run `npm run build` inside `@tamer4lynx/tamer-icons` before `pod install`, or the resource bundle may fail to compile.

## Installation

```bash
t4l add tamer-icons
# or: t4l add-core   # includes tamer-icons
```

Run **`t4l link`** after installing. **`t4l link`** also refreshes **`.tamer/tamer-components.d.ts`** and **`tsconfig.json` `include`** when **`syncTamerComponentTypes`** is enabled (default in **`tamer.config.json`**).

Peer: **`@lynx-js/types`** (for `IntrinsicElements` typings).

## TypeScript

Ambient typings ship as **`dist/icon-jsx.d.ts`** (`exports["./icon-jsx"]`, **`tamer.ambientTypeExports`**). They augment **`@lynx-js/types`** for **`<icon>`**.

**Recommended:** use **`t4l init`** / **`t4l link`** so **`.tamer/tamer-components.d.ts`** includes **`path`** references into **`node_modules/.../dist/icon-jsx.d.ts`**, and list the generated file in **`tsconfig.json`**. If you use **`src/tsconfig.json`**, include **`../.tamer/tamer-components.d.ts`** there too—see [Ambient types (CLI)](/packages/tooling/tamer-ambient-types).

**Optional:** side-effect import so typings load without relying only on the generated file:

```tsx
import '@tamer4lynx/tamer-icons'
```

## Usage

```tsx
import '@tamer4lynx/tamer-icons'

<icon
  icon="home"
  set="material"
  size={24}
  iconColor="#333"
  style={{ width: '24px', height: '24px' }}
/>
```

| Attribute | Description |
|-----------|-------------|
| `icon` | Icon name / codepoint key |
| `set` | `'material'` \| `'fontawesome'` \| `'fa'` |
| `size` | Number (often paired with `style` width/height) |
| `iconColor` | Color string |
| `style` | `ViewProps['style']` (Lynx view style) |
| `className`, `id`, … | From **`ViewProps`** — same as other Lynx views |

### Exports

| Export | Description |
|--------|-------------|
| `IconElementProps` | Props type for `<icon>` |
| `IconSet` | `'material'` \| `'fontawesome'` \| `'fa'` |
| `MATERIAL_ICONS_URL` | Material Icons font URL |
| `FONTAWESOME_SOLID_URL` | Font Awesome solid URL |
| `MATERIAL_CODEPOINTS` | Material icon codepoint map |

## See also

- [Ambient types (CLI)](/packages/tooling/tamer-ambient-types)
