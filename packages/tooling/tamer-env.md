# @tamer4lynx/tamer-env

**Rspeedy / Rsbuild plugin** for loading `.env` files during `rspeedy build` and `rspeedy dev`, and injecting values into the Lynx bundle.

## Why

The `t4l bundle` / native pipeline does not run `lynx.config.ts` with your shell environment the same way as local `npm run build`. Using **`pluginTamerEnv`** keeps env handling in one place and bakes public values into the bundle at compile time (same idea as explicit `source.define`, but driven by `.env` files).

## Install

Included in **`t4l add-core`**, or:

```bash
t4l add tamer-env
```

The package ships **`./tamer.config`**: when you use **`pluginTamer()`** in `lynx.config.ts`, **tamer-plugin** discovers `@tamer4lynx/tamer-env` in `node_modules` and merges **`pluginTamerEnv({})`** (defaults).

To **override** options (or avoid registering the plugin twice), pass **`tamerEnv`** into **`pluginTamer`** — your instance wins over the package default:

```ts
pluginTamer({
  tamerEnv: pluginTamerEnv({ root: __dirname, publicPrefix: 'TAMER_PUBLIC_' }),
})
```

To **disable** the bundled env plugin (e.g. you only use `source.define` by hand): **`pluginTamer({ tamerEnv: false })`**.

## Usage

Minimal (defaults from the package only):

```ts
import { pluginTamer } from '@tamer4lynx/tamer-plugin'

export default {
  plugins: [pluginTamer()],
}
```

With custom options (recommended when you need `defineFromEnv` or non-default `root`):

```ts
import { pluginTamer } from '@tamer4lynx/tamer-plugin'
import { pluginTamerEnv } from '@tamer4lynx/tamer-env'

export default {
  plugins: [
    pluginTamer({
      tamerEnv: pluginTamerEnv({
        root: __dirname,
      }),
    }),
  ],
}
```

### `process.env` (prefixed)

By default, only variables whose names start with **`TAMER_PUBLIC_`** are exposed as `process.env.TAMER_PUBLIC_*` in the bundle. Override the prefix with `publicPrefix`, or list extra keys with `include`.

### `__FOO__` compile-time globals

If your code uses injected globals (e.g. `__OAUTH_CLIENT_ID__`) instead of `process.env`, use **`defineFromEnv`** to map each global to an env var name, and optional **`envDefaults`** for fallbacks when a key is missing:

```ts
pluginTamerEnv({
  root: __dirname,
  defineFromEnv: {
    __OAUTH_CLIENT_ID__: 'OAUTH_CLIENT_ID',
  },
  envDefaults: {
    OAUTH_REDIRECT_URI: 'myapp://callback',
  },
})
```

Values in `source.define` in `lynx.config` still override the plugin for the same keys (merge order: plugin first, then your config).

### Files loaded

In order (later overrides earlier), when **`skipDotenv`** is not set:

1. `.env`
2. `.env.local`
3. `.env.development` or `.env.production` (from `NODE_ENV`)
4. `.env.<mode>.local`

Then **`process.env`** wins over file values for any key set in the environment (CI, shell).

## See also

- [Configuration](/guide/configuration) — `paths.lynxAdditionalBundles` for multiple bundles with `t4l start` / `t4l bundle`
- [tamer-plugin](/packages/core/tamer-plugin) — load `tamer.config` into Rsbuild
