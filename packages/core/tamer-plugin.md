# tamer-plugin

Rsbuild plugin that loads `tamer.config` from the project and workspace packages, then applies nested plugins.

## Overview

- Reads `tamer.config.json`, `tamer.config.ts`, `tamer.config.mjs`, or `tamer.config.js` from project root
- Falls back to workspace packages and `node_modules` packages that export `./tamer.config`
- Merges config; options passed to `pluginTamer()` override file config (except `false` to disable)
- Runs nested Rsbuild plugins from the merged config

## Installation

```bash
t4l add tamer-plugin
```

## Usage

```ts
import { defineConfig } from '@lynx-js/rspeedy'
import { pluginReactLynx } from '@lynx-js/react-rsbuild-plugin'
import { pluginTamer } from '@tamer4lynx/tamer-plugin'

export default defineConfig({
  plugins: [
    pluginTamer(),
    pluginReactLynx(),
  ],
})
```

## API

### pluginTamer(options?)

```ts
pluginTamer(options?: TamerPluginOptions): RsbuildPlugin
```

`TamerPluginOptions` is `Record<string, boolean | Record<string, unknown> | RsbuildPlugin>`. Values can be:

- `RsbuildPlugin` — setup is called
- `false` — disables that key from file config
- Other — overrides file config for that key

### tamer.config format

Export an object (default or `tamerDefaults`) with keys such as:

- `rsbuildConfig` — partial Rsbuild config (e.g. `source.preEntry`)
- Plugin keys — e.g. `tamerRouter` for tamer-router's plugin

Example `tamer.config.ts`:

```ts
import { tamerRouterPlugin } from '@tamer4lynx/tamer-router'

export default {
  tamerRouter: tamerRouterPlugin({
    root: './src/pages',
    output: './src/generated/_generated_routes.tsx',
    layoutFilename: '_layout.tsx',
  }),
}
```
