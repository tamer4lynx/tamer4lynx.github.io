# tamer-dev-client

Native dev client module for Tamer4Lynx — QR scan, discovery, URL persistence, reload bridge, embedded dev launcher UI.

## Overview

The dev client provides:

- **Discovery** — Find dev servers on the local network (mDNS)
- **Connect** — Enter URL or scan QR code to connect
- **Recent** — List of recently used dev server URLs
- **Reload** — Reload the Lynx bundle from the dev server
- **Compatibility check** — Validates native modules between app and project; shows modal if incompatible

### iOS: how the host’s native modules are known

The dev server’s `meta.json` lists required native modules using the same JVM-style `moduleClassName` strings as Android (from each package’s `lynx.ext.json` / `tamer.json`).

On **iOS**, Lynx does not enumerate registered modules at runtime. The host normally passes that list via `DevClientModule.attachSupportedModuleClassNames(...)` in generated `LynxInitProcessor.swift` (Tamer autolink).

Additionally, **`t4l link ios` / `t4l bundle ios` / `t4l build ios`** (via autolink) writes **`tamer-host-native-modules.json`** into `ios/<AppName>/` and adds it to **Copy Bundle Resources**. It contains `{ "moduleClassNames": [ ... ] }` using the same discovery as dev server **`meta.json`** (`nativeModules[].moduleClassName`). `DevClientModule` reads this file **only when** the set from `attachSupportedModuleClassNames` is still empty (e.g. custom `LynxInit` without the `GENERATED DEV_CLIENT_SUPPORTED` block). Re-run **`t4l link ios`** after adding or removing `@tamer4lynx/*` native packages.

**Recent tab:** Rows can show a cached icon (`data:` URL + `localStorage` when available) and a status dot: **green** when the server responds with Tamer `meta.json` and `tamerAppKey` matches the saved entry (if present); amber/gray/red for mismatch, stale meta, or offline.

When you **build with debug** (`t4l build ios -d` / `t4l build android -d`), the dev client UI is embedded and the Lynx bundle is loaded from the connected dev server. Build with **release** (`t4l build ios -r` / `t4l build android -r`) for an **unsigned** app without the dev-client shell. **Production** (`t4l build ios -p` / `t4l build android -p`) is **signed** (store-oriented) and also omits the dev client; configure signing first (`t4l signing`). See [Getting Started](/guide/getting-started) and [Commands](/reference/commands).

## Installation

```bash
t4l add tamer-dev-client
# or the full dev stack (dev-app + dev-client + dependencies):
t4l add-dev
```

The CLI resolves the **highest published semver** on npm. To add the dependency by hand, use an **explicit version** from the registry (avoid relying on **`latest`** alone). After install, run **`t4l link`** unless **`autolink`** runs it from **`postinstall`**.

## Dependencies

**Peer:** `@lynx-js/react`. **Bundled dependencies:** `@tamer4lynx/tamer-app-shell`, `tamer-insets`, `tamer-system-ui`, `tamer-plugin`, `tamer-router` (routing hooks like `useNavigate` come from **tamer-router**, which re-exports react-router—you do not need a direct `react-router` dependency for typical usage).

## Setup

Wrap your app with `DevLauncherProvider`:

```tsx
import { root } from '@lynx-js/react'
import { FileRouter } from '@tamer4lynx/tamer-router'
import { DevLauncherProvider } from '@tamer4lynx/tamer-dev-client'
import routes from '@tamer4lynx/tamer-router/generated-routes'

root.render(
  <DevLauncherProvider>
    <FileRouter routes={routes} />
  </DevLauncherProvider>
)
```

## API

### DevLauncherProvider

Context provider for the dev launcher. Pass `children` (your app tree).

### useDevLauncher()

Returns the dev launcher context. Throws if used outside `DevLauncherProvider`.

| Property | Type | Description |
|----------|------|-------------|
| `url` | `string` | Current dev server URL |
| `setUrl` | `(u: string) => void` | Set URL |
| `navigateToConnectRef` | `MutableRefObject<(() => void) \| null>` | Ref for navigating to Connect tab (e.g. after QR scan) |
| `theme` | `DevLauncherTheme \| null` | Theme from native (or null) |
| `setTheme` | `(t: DevLauncherTheme \| null) => void` | Set theme |
| `recentUrls` | `string[]` | Recently used URLs (derived from `recentEntries`) |
| `recentEntries` | `RecentEntry[]` | Recent rows: `url`, optional `label`, `iconUrl`, `tamerAppKey` |
| `recentReachability` | `Record<string, RecentReachability>` | Per-URL: `checking` \| `matched` \| `mismatch` \| `stale` \| `offline` |
| `recentRowIconSrc` | `Record<string, string>` | Cached `data:` icon URL per recent row `url` |
| `discoveredServers` | `DiscoveredServer[]` | mDNS-discovered servers |
| `removeRecentItem` | `(url: string) => void` | Remove a recent URL (native + UI state) |
| `showIncompatibleModalForUrl` | `(parsed: string) => void` | Run compatibility check and open modal if missing modules |
| `connectError` | `string` | Last connect error (e.g. unreachable URL) |
| `clearConnectError` | `() => void` | Clear `connectError` |
| `incompatibleModalVisible` | `boolean` | Whether compatibility modal is shown |
| `setIncompatibleModalVisible` | `(v: boolean) => void` | Show/hide modal |
| `incompatibleModules` | `RequiredModule[]` | Modules required by project but missing in app |
| `refreshRecent` | `() => void` | Refresh recent URLs from native |
| `connectToUrl` | `(parsed: string) => void` | Connect to a parsed URL, reload bundle |
| `openProject` | `(rawUrl: string) => void` | Parse URL and connect |
| `onSelectRecent` | `(u: string) => void` | Set URL when selecting from recent list |
| `onScanQR` | `() => void` | Trigger QR scan |
| `parseUrl` | `(input: string) => string` | Normalize URL (`tamer://` → `http://`, strip `/main.lynx.bundle`) |

### DevLauncherTheme

```ts
interface DevLauncherTheme {
  primary?: string
  primaryDark?: string
  background?: string
  surface?: string
  surfaceContainer?: string
  onSurface?: string
  onSurfaceVariant?: string
  isDark?: boolean
}
```

### resolveTheme(theme)

Merges partial theme with `FALLBACK_THEME`. Use when theme may be null.

### DiscoveredServer

```ts
type DiscoveredServer = {
  url: string
  name: string
  compatible?: boolean
  iconUrl?: string
  tamerAppKey?: string
}
```

### RecentEntry

```ts
type RecentEntry = { url: string; tamerAppKey?: string; label?: string; iconUrl?: string }
```

### RecentReachability

`'checking' | 'matched' | 'mismatch' | 'stale' | 'offline'` — see Recent tab behavior above.

### RequiredModule

```ts
type RequiredModule = { packageName: string; moduleClassName: string }
```

### devCall(method, data?, callback?)

Low-level native bridge. Methods:

| Method | Data | Callback | Description |
|--------|------|----------|-------------|
| `setDevServerUrl` | `{ url: string }` | — | Save dev server URL |
| `getDevServerUrl` | — | `(url: string) => void` | Get saved URL |
| `getRecentUrls` | — | `(urls: string[]) => void` | Get recent URLs |
| `getRecentEntries` | — | `(rows: RecentEntry[]) => void` | Get recent rows (when native supports it) |
| `removeRecentUrl` | `{ url: string }` | — | Remove one recent URL |
| `getDiscoveredServers` | — | `(servers: DiscoveredServer[]) => void` | Get mDNS servers |
| `startDiscovery` | — | — | Start mDNS discovery |
| `stopDiscovery` | — | — | Stop discovery |
| `reloadWithProjectBundle` | — | — | Reload bundle from dev server |
| `checkServerCompatibility` | `{ url: string }` | `(compatible: boolean, requiredModules: unknown) => void` | Check app vs project `meta.json` `nativeModules`. On **iOS**, native delivers one NSArray argument; **`devCall` normalizes** to the same two-arg shape as Android. Use `toRequiredModules` on the second value. |
| `scanQR` | — | — | Open QR scanner |

### toRequiredModules(raw)

Parses `checkServerCompatibility` callback result into `RequiredModule[]`.

## Built-in pages

The dev client ships with Connect, Recent, and Discover pages. The layout uses `Tabs` with `titleForPath` for dynamic titles. Events:

- `devclient:scanResult` — QR scan result (payload: `{ url }`)
- `devclient:discoveredServers` — mDNS servers (payload: `{ servers }`)

## Standard module pattern

`tamer-dev-client` is a standard tamer module. When installed, `t4l sync android` (or `t4l sync`) reads host templates from the package (`android/templates/`, `ios/templates/`) and applies them to wire the dev client into your host app. The templates use `{{PACKAGE_NAME}}` and `{{APP_NAME}}` placeholders. (Sync is supported for Android; iOS dev client is synced via the iOS create/sync flow.)

## Build behavior

- **Debug build** (`t4l build ios -d` / `t4l build android -d`): If `@tamer4lynx/tamer-dev-client` is in your app's dependencies, the dev client is embedded. Your app becomes a dev app (QR scan, HMR).
- **Release build** (`t4l build ios -r` / `t4l build android -r`): The dev client is not included. Use for production or store builds.

## tamer-dev-app

**tamer-dev-app** is a standalone package in the ecosystem that bundles **tamer-dev-client** for a reference dev launcher. In practice you still **build** that app (or your own app with **tamer-dev-client**) with **`t4l build … -d`** after **`t4l link`**, so the native side matches **your** linked modules—the same reason there is not yet a single prebuilt “install everywhere” HMR app on the store.

A **generalized** dev launcher (one binary for typical workflows without per-project builds) is **planned**. Until then, treat your **debug** build as the HMR shell tailored to your **`node_modules`** and autolink output.
