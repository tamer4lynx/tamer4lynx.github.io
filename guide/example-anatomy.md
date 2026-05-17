# Example App Anatomy

`packages/example` is the showcase Lynx app for Tamer4Lynx: file-based routing with three top-level sections (tabs, connectors demos, an M3 component playground), a stack of native-module demos, theming via a Recoil-backed palette, and a switch in `index.tsx` for testing alternative router shapes.

This page walks through what's actually in the package today.

---

## Directory structure

```
packages/example/
├── lynx.config.ts          # Rspeedy: pluginTamer, pluginQRCode, pluginReactLynx, pluginTamerEnv, pluginTypeCheck; workspace aliases; OAuth define
├── package.json            # Depends on every @tamer4lynx/* package; some via file: paths in the monorepo
├── tsconfig.json
├── biome.json              # Biome lint/format
├── vitest.config.ts        # Vitest + jsdom
├── server.t.s              # Optional dev helper (`bun run server`)
├── .env / .env.example     # OAuth + transport env vars
├── intrinsic-element.d.ts  # Augments Lynx JSX for input, textarea, svg
├── src/
│   ├── index.tsx                       # Entry — switches between Stack / ExampleStack / FileRouterApp via testExample
│   ├── App.tsx                         # Standalone (non-router) entry; reachable when testExample=0/1
│   ├── App.css
│   ├── Stack.tsx                       # Manual stack used by App.tsx
│   ├── example_stack.tsx               # Alternative manual stack
│   ├── file-router-app.tsx             # FileRouter wired with providerConnector + DetailsRecoilProvider
│   ├── tamer-app-routes.tsx            # Re-exports TamerAppRoutes / TAMER_KNOWN_PATHS from tamer-router/generated-app-routes
│   ├── examplePalette.ts               # useExamplePalette() — central palette + theme bootstrap
│   ├── useSyncNavigationChrome.ts      # Keeps status / navigation bar colors in sync with palette
│   ├── connectors-state.tsx            # Provider registry for connector demos (zustand, redux, recoil, …)
│   ├── connectors-demo-page.tsx        # Shared scaffolding for each /connectors/* demo
│   ├── details-recoil-state.tsx        # Recoil provider used by FileRouterApp
│   ├── demo-store.ts                   # Shared store data fed to connectors
│   ├── oauth-config.ts                 # OAuth client config for the `native/auth` route
│   ├── tamer-icons-fonts.generated.ts  # Generated font manifest for tamer-icons
│   ├── rspeedy-env.d.ts                # JSX typings for app-bar, icon
│   ├── polyfills/                      # url-search-params and friends
│   ├── assets/                         # tamer / lynx / react logos
│   ├── fonts/                          # Material Icons + Font Awesome .ttf (used by tamer-icons-fonts.generated.ts)
│   ├── __tests__/                      # vitest specs
│   └── pages/
│       ├── index.tsx                   # `/` — emits a Slot so the file router resolves into the appropriate group layout
│       ├── not_layout.tsx              # Reserved name; file router treats it as a non-layout page
│       ├── tabs/
│       │   ├── _layout.tsx             # Bottom Tab bar layout (Tab / Tab.Screen) + palette-driven status/nav chrome
│       │   ├── index.tsx               # Home
│       │   ├── about.tsx
│       │   ├── insets.tsx              # tamer-insets demo + <textarea>
│       │   ├── screen.tsx              # tamer-screen
│       │   └── secure.tsx              # tamer-secure-store + tamer-biometric
│       ├── m3/
│       │   ├── _layout.tsx             # Stack with pathPrefix /m3
│       │   ├── index.tsx               # M3 component gallery
│       │   └── nav.tsx                 # M3 navigation patterns
│       ├── connectors/
│       │   ├── _layout.tsx             # Stack with pathPrefix /connectors
│       │   ├── index.tsx               # Demo index
│       │   ├── zustand.tsx       + zustand-detail.tsx
│       │   ├── redux.tsx         + redux-detail.tsx
│       │   ├── tanstack-query.tsx + tanstack-query-detail.tsx
│       │   ├── apollo.tsx        + apollo-detail.tsx
│       │   ├── swr.tsx           + swr-detail.tsx
│       │   ├── jotai.tsx         + jotai-detail.tsx
│       │   ├── i18next.tsx       + i18next-detail.tsx
│       │   ├── recoil.tsx        + recoil-detail.tsx
│       │   └── theme.tsx         + theme-detail.tsx
│       └── native/
│           ├── _layout.tsx             # Stack with pathPrefix /native + palette chrome
│           ├── index.tsx               # Hub linking into the demos below
│           ├── auth.tsx                # tamer-auth (OAuth / PKCE)
│           ├── linking.tsx             # tamer-linking
│           ├── browser.tsx             # tamer-display-browser
│           ├── storage.tsx             # tamer-local-storage
│           ├── transports.tsx          # tamer-transports
│           ├── webview.tsx             # tamer-webview (<webview>)
│           └── details/
│               ├── _layout.tsx
│               ├── [id].tsx            # Dynamic route, recoil-driven detail screen
│               └── edit.tsx
```

---

## Entry point — three router shapes via a switch

`src/index.tsx` lets you flip between three router setups by editing `testExample`:

```tsx
import 'url-search-params-polyfill'
import '@lynx-js/react/debug'
import '@tamer4lynx/tamer-transports/lynx'
import { root } from '@lynx-js/react'

import Stack from './Stack.jsx'
import ExampleStack from './example_stack.jsx'
import { FileRouterApp } from './file-router-app.js'

const testExample: number = -1

let component: JSX.Element
switch (testExample) {
  case 0:
    component = <Stack />
    break
  case 1:
    component = <ExampleStack />
    break
  default:
    component = <FileRouterApp />
    break
}

root.render(component)

if (import.meta.webpackHot) {
  import.meta.webpackHot.accept()
}
```

- **`Stack` / `ExampleStack`** — manual, hand-written router shapes used to validate `tamer-router` primitives without the file-based plugin.
- **`FileRouterApp`** — the default and recommended setup. Pulls routes from the tamer-router plugin's generated tree.
- **`@tamer4lynx/tamer-transports/lynx`** — required side-effect import that polyfills `fetch`, `WebSocket`, `EventSource` for HMR + native networking.

---

## FileRouterApp — connectors and Recoil

`src/file-router-app.tsx` wires the file router with a registry of provider connectors so each `/connectors/*` demo gets its own state library at the top of the tree:

```tsx
import { FileRouter } from '@tamer4lynx/tamer-router'
import '@tamer4lynx/tamer-icons'
import './tamer-app-routes.js'
import { DetailsRecoilProvider, detailsRecoilProviderConnector } from './details-recoil-state.js'
import { connectorsProviderRegistry } from './connectors-state.js'
import { useExamplePalette } from './examplePalette.js'

const bridgeProviderConnector = [
  detailsRecoilProviderConnector,
  ...connectorsProviderRegistry,
]

export function FileRouterApp() {
  useExamplePalette()
  return (
    <DetailsRecoilProvider>
      <FileRouter providerConnector={bridgeProviderConnector} />
    </DetailsRecoilProvider>
  )
}
```

- **`useExamplePalette()`** — pulls a Recoil-backed palette + theme; the layouts read it via `useSyncNavigationChrome` to color the status / nav bars.
- **`providerConnector`** — array of provider factories the router mounts at the right place in the tree (one per demo).
- **`tamer-app-routes.tsx`** — re-exports `TamerAppRoutes` + `TAMER_KNOWN_PATHS` from the auto-generated route module so other files can reference paths without stringly-typed URLs.

---

## lynx.config.ts

Wires Rspeedy with Tamer plugins and workspace aliases:

| Section | Purpose |
|---------|---------|
| `resolve.alias` | Points `@tamer4lynx/tamer-app-shell`, `tamer-navigation`, `tamer-router` (and a few others) at local `../<pkg>/src` so monorepo dev sees source, not dist; also aliases `react` → `@lynx-js/react/compat`. |
| `resolve.dedupe` | Forces single copies of `@react-navigation/core` + `@react-navigation/routers`. |
| `source.define` | Injects OAuth env vars (`OAUTH_CLIENT_ID`, etc.) loaded from `.env`. |
| `plugins` | `pluginTamer({ tamerRouter })`, `pluginReactLynx`, `pluginQRCode` (fullscreen QR scheme), `pluginTamerEnv`, `pluginTypeCheck` (when `TAMER_TYPE_CHECK=1`). |

`pluginTamer({ tamerRouter })` is what generates the route tree the file router consumes; `tamerRouter` is created with `tamerRouterPlugin({ root: './src/pages', output: 'src/generated-routes.tsx', layoutFilename: '_layout.tsx' })`.

---

## Routing

### `pages/index.tsx`

Top-level entry just emits a `<Slot />` so the file router resolves the first matching child group:

```tsx
import { Slot } from '@tamer4lynx/tamer-router'

export default function IndexEntry() {
  return <Slot />
}
```

### Tab bar — `pages/tabs/_layout.tsx`

```tsx
import { Tab } from '@tamer4lynx/tamer-router'
import { useSystemUI } from '@tamer4lynx/tamer-system-ui'
import { useExamplePalette } from '../../examplePalette.js'
import { useSyncNavigationChrome } from '../../useSyncNavigationChrome.js'

export default function Layout() {
  const { setStatusBar, setNavigationBar } = useSystemUI()
  const p = useExamplePalette()
  const dark = p.theme?.isDark !== false
  useSyncNavigationChrome(setStatusBar, setNavigationBar, p.barBackground, dark)

  return (
    <Tab>
      <Tab.Screen name="index" options={{ title: 'Tamer4Lynx', icon: 'home' }} />
      <Tab.Screen name="about" options={{ title: 'About', icon: 'info' }} />
      <Tab.Screen name="insets" options={{ title: 'Insets', icon: 'settings' }} />
      <Tab.Screen name="screen" options={{ title: 'Screen', icon: 'list' }} />
      <Tab.Screen name="secure" options={{ title: 'Secure Number', icon: 'lock' }} />
    </Tab>
  )
}
```

Note: this app uses **`Tab`** (singular) — `Tab.Screen` for each route. Earlier APIs exposed it as `Tabs` / `Tabs.Screen`; the example tracks the current export.

### Stack groups — `pages/native/_layout.tsx` (and `m3`, `connectors`)

Stacks declare a `pathPrefix` so nested files resolve relative to the group:

```tsx
import { Stack } from '@tamer4lynx/tamer-router'

export default function DevLayout() {
  // …palette + chrome sync omitted
  return (
    <Stack pathPrefix="/native">
      <Stack.Screen name="index" options={{ title: 'Native Tests' }} />
      <Stack.Screen name="auth" options={{ title: 'OAuth' }} />
      <Stack.Screen name="linking" options={{ title: 'tamer-linking' }} />
      <Stack.Screen name="browser" options={{ title: 'tamer-display-browser' }} />
      <Stack.Screen name="storage" options={{ title: 'tamer-local-storage' }} />
      <Stack.Screen name="webview" options={{ title: 'tamer-webview' }} />
      <Stack.Screen name="transports" options={{ title: 'tamer-transports' }} />
    </Stack>
  )
}
```

The `connectors/_layout.tsx` follows the same shape with one `Stack.Screen` per state-library demo (zustand, redux, tanstack-query, apollo, swr, jotai, i18next, recoil, theme — each with a matching `-detail` screen). `m3/_layout.tsx` covers the M3 component playground (`index`, `nav`).

---

## Pages and packages

| Route group | Pages | Packages exercised |
|-------------|-------|--------------------|
| `tabs/` | `index`, `about`, `insets`, `screen`, `secure` | tamer-router, tamer-icons, tamer-system-ui, tamer-insets, tamer-screen, tamer-secure-store, tamer-biometric, jiggle |
| `m3/` | `index`, `nav` | M3-styled components built on Lynx primitives |
| `connectors/` | `zustand`, `redux`, `tanstack-query`, `apollo`, `swr`, `jotai`, `i18next`, `recoil`, `theme` (each + `-detail`) | Demonstrates one external state / data library per route. Providers are registered via `connectorsProviderRegistry`. |
| `native/` | `index`, `auth`, `linking`, `browser`, `storage`, `transports`, `webview`, `details/[id]`, `details/edit` | tamer-auth (OAuth), tamer-linking, tamer-display-browser, tamer-local-storage, tamer-transports, tamer-webview, plus a Recoil-backed dynamic `details/[id]` route |

---

## Assets and fonts

- **Images** — `src/assets/` PNGs imported by the home screen (`tamer-logo`, `lynx-logo`, `react-logo`).
- **Fonts** — `tamer-icons-fonts.generated.ts` maps Material Icons + Font Awesome (solid, regular, brands) CDN URLs to local `./fonts/*.ttf` for [tamer-icons](/packages/ui/tamer-icons). The tamer-icons build downloads + caches these on first run.

---

## TypeScript

- **`intrinsic-element.d.ts`** — extends Lynx's JSX namespace with `<input>`, `<textarea>`, and `<svg content="…" />`.
- **`rspeedy-env.d.ts`** — declares `app-bar` + `icon` for JSX completeness.
- Generated route types land in `src/generated-routes.tsx` / `routeTree.gen.ts` from the tamer-router plugin.

---

## Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `dev` | `rspeedy dev` | Dev server with HMR |
| `build` | `rspeedy build` | Production bundle |
| `build:native` | `rspeedy build` + copy bundle into the local Android assets dir | Helper for direct on-device testing inside the monorepo |
| `preview` | `rspeedy preview` | Web preview |
| `test` | `vitest run` | Unit tests |
| `check` | `biome check --write` | Lint + format check |
| `format` | `biome format --write` | Format |
| `server` | `bun run server.t.s` | Optional dev helper for the transport demos |

From the monorepo root, use `t4l start` and `t4l build ios` / `t4l build android` to run the dev server and build the native shells.

---

## Next steps

- [Getting Started](/guide/getting-started) — Set up a new project (prebuilt dev app or DIY)
- [Native Modules & Elements](/guide/native-modules) — Author a Tamer-compatible native package
- [Configuration](/guide/configuration) — `tamer.config.json`, `lynx.config.ts`, `lynx.ext.json`
- [tamer-router](/packages/core/tamer-router) — File-based routing details
