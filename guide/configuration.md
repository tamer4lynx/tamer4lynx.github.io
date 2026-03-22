# Configuration Reference

Tamer4Lynx uses three configuration files. Each serves a different purpose.

---

## 1. Project config: `tamer.config.json`

**Location:** Project root (where you run `t4l`).

**Purpose:** Defines the host native app: Android/iOS project paths, app identity, Lynx bundle location, dev server, and autolinking. Used by the CLI for `t4l create android`, `t4l create ios`, `t4l link`, `t4l build`, `t4l start`, etc.

Create it with `t4l init` or `t4l`.

### Signing and production builds (`-p`)

**Store-ready / signed** native builds use **`t4l build [platform] -p`** (or `--production`). That is separate from **`-r`** (unsigned release without the dev client).

1. Run **`t4l signing`**, **`t4l signing android`**, or **`t4l signing ios`** to write signing-related fields into `tamer.config.json` (and optionally append keystore password env var names to `.env` / `.env.local` on Android). See the full behavior in [Commands](/reference/commands) under **`t4l signing`** and **`t4l build`**.
2. **`t4l build android -p`** or **`t4l build ios -p`** — each command targets one platform and requires signing for that platform only.

On **iOS**, **`-p`** builds for **`iphoneos`** (not the simulator). For App Store submission you still typically archive and export an IPA in Xcode; the CLI documents that limitation in [Commands](/reference/commands).

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `android` | object | Android app configuration |
| `android.appName` | string | Display name of the Android app |
| `android.packageName` | string | Application ID (e.g. `com.example.myapp`) |
| `android.sdk` | string | Path to Android SDK (supports `~`, `$ANDROID_HOME`) |
| `android.deepLinks` | array | Deep link configs for intent filters. Each: `{ scheme, host?, pathPrefix?, activity? }` |
| `android.abiFilters` | string[] | NDK ABI filters (default: `armeabi-v7a`, `arm64-v8a`) |
| `ios` | object | iOS app configuration |
| `ios.appName` | string | Display name of the iOS app |
| `ios.bundleId` | string | Bundle identifier (e.g. `com.example.MyApp`) |
| `ios.urlSchemes` | array | URL schemes for deep linking. Each: `{ scheme, role? }` |
| `ios.architectures` | string[] | Build architectures (default: `arm64`) |
| `lynxProject` | string | Path to Lynx project (e.g. `packages/example`). Omit for single-folder layout (Lynx app is project root) or to auto-discover via workspaces |
| `paths` | object | Override default paths |
| `paths.androidDir` | string | Android project directory (default: `android`) |
| `paths.iosDir` | string | iOS project directory (default: `ios`) |
| `paths.lynxProject` | string | Same as top-level `lynxProject` |
| `paths.lynxBundleRoot` | string | Bundle output dir inside Lynx project (default: `dist`) |
| `paths.lynxBundleFile` | string | Bundle filename (default: `main.lynx.bundle`) |
| `dev` | object | Dev mode configuration (optional). Dev client inclusion is now controlled by build type: **debug** (`-d`) embeds the dev client when `@tamer4lynx/tamer-dev-client` is installed; **release** (`-r`) omits it. |
| `dev.mode` | string | `standalone` \| `embedded` \| `off`. Prefer `-d` / `-r` for dev client behavior when applicable. |
| `devServer` | object | Dev server for HMR (host, port). Used when dev client is present. |
| `devServer.host` | string | Host for native app to connect (e.g. `10.0.2.2` for Android emulator) |
| `devServer.port` | number | Dev server port (default: 3000) |
| `devServer.httpPort` | number | Alternative to `port` |
| `icon` | string \| object | App icon: path string, or `{ source?, android?, ios?, androidAdaptive? }`. **`androidAdaptive`** (Android 8+): `{ foreground, background? }` or `{ foreground, backgroundColor? }`. Optional layout: **`foregroundScale`**: 0–1 uniform shrink (e.g. `0.62`); **`foregroundPadding`**: extra inset as number (% per side), `"8dp"`, `"10%"`, or `{ left, top, right, bottom }`. Scale and padding are merged into `<layer-list>` insets. `t4l build` / `t4l bundle` refreshes icons. |
| `autolink` | boolean | When `true`, `t4l link` runs after `npm install` (if postinstall is configured) |
| `syncTamerComponentTypes` | boolean | When not `false` (default), `t4l init` / `t4l link` generate `.tamer/tamer-components.d.ts` and update `tsconfig` `include`. Set to `false` to skip |

### iOS autolink artifacts

Besides `Podfile` and `LynxInitProcessor.swift`, **`t4l link ios`** (and any command that runs iOS autolink) may generate:

| File | When | Purpose |
|------|------|---------|
| `tamer-host-native-modules.json` | `@tamer4lynx/tamer-dev-client` is in `node_modules` | Lists `moduleClassName` values for dev-client **compatibility** with dev server `meta.json`; copied into the app bundle as a resource |

Commit this JSON (or re-run `t4l link ios` on CI) so the app stays aligned with linked native packages.

### Examples

**Single-folder layout** (Lynx app, config, android/, ios/ in one directory — omit `lynxProject`):

```json
{
  "android": { "appName": "MyApp", "packageName": "com.example.myapp", "sdk": "~/Library/Android/sdk" },
  "ios": { "appName": "MyApp", "bundleId": "com.example.MyApp" },
  "paths": { "androidDir": "android", "iosDir": "ios" },
  "autolink": true
}
```

**Monorepo layout** (Lynx app in a package):

```json
{
  "android": { "appName": "MyApp", "packageName": "com.example.myapp", "sdk": "~/Library/Android/sdk" },
  "ios": { "appName": "MyApp", "bundleId": "com.example.MyApp" },
  "lynxProject": "packages/example",
  "paths": { "androidDir": "android", "iosDir": "ios" },
  "autolink": true
}
```

**Android adaptive icon** (light logo on dark): set `foreground` to the logo asset and `backgroundColor` instead of a background image:

```json
{
  "icon": {
    "source": "packages/example/src/assets/logo.png",
    "androidAdaptive": {
      "foreground": "packages/example/src/assets/logo.png",
      "backgroundColor": "#000000"
    }
  }
}
```

If the logo is clipped by the launcher mask, shrink and/or pad the foreground layer:

```json
{
  "androidAdaptive": {
    "foreground": "packages/example/src/assets/logo.png",
    "backgroundColor": "#000000",
    "foregroundScale": 0.62,
    "foregroundPadding": 6
  }
}
```

`foregroundScale=0.62` adds `((1−0.62)/2)*100 ≈ 19%` inset on each side. `foregroundPadding=6` adds another 6% per side. Both are merged into `android:left/top/right/bottom` on a `<layer-list>` item — no `ScaleDrawable` quirks.

Keep `source` (and optional `ios`) so fallback launcher mipmaps and the iOS app icon still resolve from the same file.

---

## 2. Component config: `tamer.config.ts` (or `.mjs` / `.js`)

**Location:** Project root or in packages that export `./tamer.config`.

**Purpose:** Rsbuild plugin configuration. Used by **tamer-plugin** to load plugin instances (e.g. tamer-router, file-based routing). This is **not** the same as `tamer.config.json` — it is a TypeScript/JS module that exports plugin defaults.

**Used by:** `pluginTamer()` in your `lynx.config.ts`. Do not import `tamer.config` in `lynx.config`; use `pluginTamer()` so it loads `tamer.config` internally.

### Format

Export an object (default or `tamerDefaults`) with keys such as:

- `rsbuildConfig` — partial Rsbuild config (e.g. `source.preEntry`)
- Plugin keys — e.g. `tamerRouter` for tamer-router's plugin

### Example

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

### Discovery order

1. Project root: `tamer.config.ts` → `tamer.config.mjs` → `tamer.config.js`
2. Workspace packages that export `./tamer.config`
3. `node_modules` packages that export `./tamer.config`

See [tamer-plugin](/packages/core/tamer-plugin) for details.

---

## 3. Extension config: `lynx.ext.json`

**Location:** Inside each native extension package (e.g. `node_modules/@tamer4lynx/jiggle/lynx.ext.json`).

**Purpose:** Declares how the extension integrates with Android and iOS. Used by the autolinker (`t4l link`) to add the native module to Gradle and Podfile, register modules/elements, and sync permissions.

Tamer4Lynx follows the [Lynx Autolink RFC](https://github.com/lynx-family/lynx/discussions/2653). If you want to help improve the RFC or lynx.ext.json format, see the [RFC discussion](https://github.com/lynx-family/lynx/discussions/2653).

### Format (RFC `platforms`)

```json
{
  "platforms": {
    "android": { ... },
    "ios": { ... },
    "web": {}
  }
}
```

### Android fields

| Field | Type | Description |
|-------|------|-------------|
| `packageName` | string | Java/Kotlin package (e.g. `com.example.mymodule`) |
| `moduleClassName` | string | Native module class (e.g. `com.example.mymodule.MyModule`) |
| `moduleClassNames` | string[] | Multiple modules |
| `sourceDir` | string | Path to Android source (default: `android`) |
| `elements` | object | Custom elements: `{ "tag-name": "ClassName" }` |
| `permissions` | string[] | Android permissions to merge into manifest |
| `attachHostView` | boolean | Attach module to host view lifecycle |
| `activityPatches` | object | Activity lifecycle hooks: `onCreate`, `onResume`, etc. |

### iOS fields

| Field | Type | Description |
|-------|------|-------------|
| `podspecPath` | string | Path to podspec directory (e.g. `ios/mymodule`) |
| `moduleClassName` | string | Native module class name |
| `moduleClassNames` | string[] | Multiple modules |
| `elements` | object | Custom elements: `{ "tag-name": "ClassName" }` |
| `iosPermissions` | object | Info.plist keys: `{ "NSCameraUsageDescription": "..." }` |
| `attachHostView` | boolean | Attach module to host view lifecycle |

### Example

```json
{
  "platforms": {
    "android": {
      "packageName": "com.nanofuxion.vibration",
      "moduleClassName": "com.nanofuxion.vibration.JiggleModule",
      "sourceDir": "android"
    },
    "ios": {
      "podspecPath": "ios/jiggle",
      "moduleClassName": "JiggleModule"
    },
    "web": {}
  }
}
```

### `tamer.json` (flat format)

Tamer4Lynx also supports `tamer.json` (flat format). Prefer `lynx.ext.json` with `platforms` for new extensions.

---

## Summary

| Config | Location | Used by | Purpose |
|--------|----------|---------|---------|
| `tamer.config.json` | Project root | t4l CLI | Host app: Android/iOS identity, paths, dev server, autolink |
| `tamer.config.ts` | Project root, packages | tamer-plugin | Rsbuild plugins (routing, etc.) |
| `lynx.ext.json` | Extension package | t4l link (autolinker) | Native module registration per platform |
