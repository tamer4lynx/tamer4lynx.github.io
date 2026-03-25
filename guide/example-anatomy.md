# Example App Anatomy

The `packages/example` app demonstrates a full Tamer4Lynx setup: file-based routing, tabs, native modules, and platform APIs. This page walks through its structure.

---

## Directory structure

```
packages/example/
├── lynx.config.ts          # Rspeedy: pluginTamer, QR, ReactLynx, type-check; aliases; OAuth define
├── package.json
├── tsconfig.json
├── biome.json              # Biome lint/format
├── vitest.config.ts        # Vitest + jsdom
├── .env.example            # OAuth vars for native/auth
├── intrinsic-element.d.ts  # input, textarea, svg typings
├── src/
│   ├── index.tsx           # Entry: FileRouter + generated routes + dev HMR accept
│   ├── App.tsx             # Standalone entry (not used with router)
│   ├── App.css
│   ├── rspeedy-env.d.ts    # JSX typings for app-bar, icon
│   ├── oauth-config.ts     # OAuth config (native/auth)
│   ├── tamer-icons-fonts.generated.ts  # Font manifest for tamer-icons
│   ├── assets/
│   │   ├── tamer-logo.png
│   │   ├── lynx-logo.png
│   │   └── react-logo.png
│   ├── fonts/              # Material Icons, Font Awesome (.ttf — used by tamer-icons-fonts.generated.ts)
│   │   ├── MaterialIcons-Regular.ttf
│   │   ├── fa-solid-900.ttf
│   │   ├── fa-regular-400.ttf
│   │   └── fa-brands-400.ttf
│   ├── __tests__/
│   │   └── index.test.tsx
│   └── pages/
│       ├── _layout.tsx     # Root layout: Tabs + useSystemUI
│       ├── index.tsx       # Home tab
│       ├── about.tsx
│       ├── insets.tsx      # tamer-insets, textarea
│       ├── screen.tsx      # tamer-screen
│       ├── secure.tsx      # tamer-secure-store, tamer-biometric
│       └── native/
│           ├── _layout.tsx  # Stack layout for native tests
│           ├── index.tsx   # Links into OAuth, linking, browser, storage, webview
│           ├── auth.tsx    # tamer-auth (OAuth)
│           ├── linking.tsx  # tamer-linking
│           ├── browser.tsx  # tamer-display-browser
│           ├── storage.tsx # tamer-local-storage (Web localStorage API)
│           └── webview.tsx # tamer-webview (<webview>)
```

---

## Entry point

`src/index.tsx` bootstraps the app with [tamer-router](/packages/core/tamer-router) and generated routes:

```tsx
import '@lynx-js/react/debug'
import '@tamer4lynx/tamer-transports/lynx'
import { root } from '@lynx-js/react'
import { FileRouter } from '@tamer4lynx/tamer-router'

import routes from '@tamer4lynx/tamer-router/generated-routes'

root.render(<FileRouter routes={routes} />)

if (import.meta.webpackHot) {
  import.meta.webpackHot.accept()
}
```

- **tamer-transports/lynx** — Polyfills for fetch, WebSocket, EventSource (needed for HMR and native networking).
- **FileRouter** — Renders the file-based route tree from `generated-routes`.
- **generated-routes** — Produced by tamer-router plugin from `src/pages/`.
- **`webpackHot.accept()`** — Keeps the dev client alive across HMR updates when using Rspeedy dev.

---

## lynx.config.ts

Configures Rspeedy with Tamer plugins and workspace aliases:

| Section | Purpose |
|--------|---------|
| `source.alias` | Points some `@tamer4lynx/*` packages at local `../package/src` for monorepo dev |
| `source.define` | Injects OAuth env vars (`OAUTH_CLIENT_ID`, etc.) from `.env` |
| `plugins` | `pluginTamer`, `pluginQRCode` (fullscreen QR schema), `pluginReactLynx`, `pluginTypeCheck` (when `TAMER_TYPE_CHECK=1`) |

`pluginTamer()` discovers `tamer.config` from workspace packages (tamer-router provides defaults) and applies the file-based routing plugin.

---

## Routing

### Root layout (`pages/_layout.tsx`)

Uses **Tabs** from tamer-router and **useSystemUI** for status/nav bar:

```tsx
import { useEffect } from '@lynx-js/react'
import { Tabs } from '@tamer4lynx/tamer-router'
import { useSystemUI } from '@tamer4lynx/tamer-system-ui'

export default function Layout() {
  const { setStatusBar, setNavigationBar } = useSystemUI()

  useEffect(() => {
    setStatusBar({ color: '#fff', style: 'light' })
    setNavigationBar({ color: '#fff', style: 'light' })
  }, [])

  return (
    <Tabs
      screenOptions={{
        headerStyle: { backgroundColor: '#555' },
        tabBarStyle: { backgroundColor: '#555' },
      }}
    >
      <Tabs.Screen name="index" path="/" options={{ title: 'Tamer4Lynx', icon: 'home', label: 'Home' }} />
      <Tabs.Screen name="about" path="/about" options={{ title: 'About', icon: 'info', label: 'About' }} />
      <Tabs.Screen name="insets" path="/insets" options={{ title: 'Insets', icon: 'fit_screen', label: 'Insets' }} />
      <Tabs.Screen name="screen" path="/screen" options={{ title: 'Screen', icon: 'list', label: 'Screen' }} />
      <Tabs.Screen name="secure" path="/secure" options={{ title: 'Secure Number', icon: 'lock', label: 'Secure' }} />
    </Tabs>
  )
}
```

### Nested Stack (`pages/native/_layout.tsx`)

The `/native` route group uses a **Stack** for OAuth, linking, browser, persistent storage, and webview:

```tsx
import { Stack } from '@tamer4lynx/tamer-router'

export default function DevLayout() {
  return (
    <Stack screenOptions={{ headerStyle: { backgroundColor: '#555' } }}>
      <Stack.Screen name="index" path="/native" options={{ title: 'Native Tests' }} />
      <Stack.Screen name="auth" path="/native/auth" options={{ title: 'OAuth' }} />
      <Stack.Screen name="linking" path="/native/linking" options={{ title: 'tamer-linking' }} />
      <Stack.Screen name="browser" path="/native/browser" options={{ title: 'tamer-display-browser' }} />
      <Stack.Screen name="storage" path="/native/storage" options={{ title: 'tamer-local-storage' }} />
      <Stack.Screen name="webview" path="/native/webview" options={{ title: 'tamer-webview' }} />
    </Stack>
  )
}
```

---

## Pages and packages

| Page | Packages used | Purpose |
|------|----------------|---------|
| `index.tsx` | tamer-router, tamer-icons, jiggle | Home: logo tap, navigation, `<icon>` demo (`import '@tamer4lynx/tamer-icons'`) |
| `insets.tsx` | tamer-insets | Safe area, keyboard insets, `<textarea>` |
| `screen.tsx` | tamer-screen | Screen, SafeArea, AvoidKeyboard |
| `secure.tsx` | tamer-secure-store, tamer-biometric | Encrypted async key-value + biometric gate |
| `native/auth.tsx` | tamer-auth | OAuth 2.0 / PKCE flow |
| `native/linking.tsx` | tamer-linking | Deep linking |
| `native/browser.tsx` | tamer-display-browser | In-app browser for OAuth |
| `native/storage.tsx` | tamer-local-storage | Web `localStorage`–style sync API (`localStorage.getItem` / `setItem`) |
| `native/webview.tsx` | tamer-webview | Native `<webview>` (WKWebView / Android WebView); typings via `.tamer/tamer-components.d.ts` |

---

## Assets and fonts

- **Images** — `src/assets/` PNGs imported in home (`tamer-logo`, `lynx-logo`, `react-logo`).
- **Fonts** — `tamer-icons-fonts.generated.ts` maps CDN URLs to local `./fonts/*.ttf` for [tamer-icons](/packages/ui/tamer-icons) (Material Icons, Font Awesome). Run the tamer-icons package build if those files are missing locally.

---

## TypeScript

- **intrinsic-element.d.ts** — Extends Lynx types for `<input>`, `<textarea>`, and `<svg content="..." />`.
- **rspeedy-env.d.ts** — Declares `app-bar`, `icon` for JSX.

---

## Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `dev` | `rspeedy dev` | Dev server with HMR |
| `build` | `rspeedy build` | Production bundle |
| `build:native` | `rspeedy build` + copy bundle (local path in repo) | Example helper for Android assets |
| `preview` | `rspeedy preview` | Web preview |
| `test` | `vitest run` | Unit tests |
| `check` | `biome check --write` | Lint + format check |
| `format` | `biome format --write` | Format |
| `server` | `bun run server.t.s` | Optional dev helper |

From the monorepo root, use `t4l start` and `t4l build ios` / `t4l build android` to run the dev server and build the native app.

---

## Next steps

- [Getting Started](/guide/getting-started/) — Set up a new project
- [Configuration](/guide/configuration) — tamer.config.json, lynx.config.ts
- [tamer-router](/packages/core/tamer-router) — File-based routing details
