# tamer-router

File-based routing for Lynx with React 17 and react-router 6.

## Overview

- Rsbuild plugin: scans a folder and generates a route tree
- Conventions: `index` → index route, `[param]` → dynamic segment, `_layout.tsx` → layout wrapper
- **Stack** and **Tabs** layouts with AppBar, TabBar, Content (via tamer-app-shell)
- `useTamerRouter()` / `useTamerNavigate()` for stack-aware navigation (`push`, `replace`, `back`, `canGoBack`)
- **System back (Android hardware back / iOS gesture):** the native **`TamerRouterNativeModule`** emits a **`tamer-router:back`** event on **`GlobalEventEmitter`**. `FileRouter` subscribes and runs **screen-level back handlers first** (`useBackHandler` / `usePreventBack`); if none consume the event (`return true`), the router pops the stack when `canGoBack()` is true. The JS side calls **`NativeModules.TamerRouterNativeModule.didHandleBack(consumed)`** so native can finish transitions / snapshot overlays. See **Hardware back** below.

## Installation

```bash
t4l add tamer-router
```

**Peers:** **`react-router@^6`** — add it if your package manager does not install peers automatically (`npm install react-router@6`, etc.). **tamer-app-shell** is a dependency of this package for layouts; **`t4l add-core`** also installs app-shell. Run **`t4l link`** after adding packages.

## Setup

### 1. Lynx config (Rspeedy)

Use **tamer-plugin** so the default tamer.config from tamer-router is applied:

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

Or add **tamerRouterPlugin** directly:

```ts
import { tamerRouterPlugin } from '@tamer4lynx/tamer-router'

tamerRouterPlugin({
  root: './src/pages',
  output: './src/generated/_generated_routes.tsx',
  srcAlias: '@/',
  layoutFilename: '_layout.tsx',
})
```

### 2. Entry point

**Option A: Simple FileRouter**

```tsx
import { root } from '@lynx-js/react'
import { FileRouter } from '@tamer4lynx/tamer-router'
import routes from '@tamer4lynx/tamer-router/generated-routes'

root.render(<FileRouter routes={routes} />)
```

**Option B: Tabs layout (recommended)**

Use `Tabs` in `_layout.tsx` for AppBar + TabBar:

```tsx
import { Tabs } from '@tamer4lynx/tamer-router'
import { useSystemUI } from '@tamer4lynx/tamer-system-ui'

export default function Layout() {
  const { setStatusBar, setNavigationBar } = useSystemUI()

  useEffect(() => {
    setStatusBar({ color: '#fff', style: 'light' })
    setNavigationBar({ color: '#fff', style: 'light' })
  }, [])

  return (
    <Tabs screenOptions={{ headerStyle: { backgroundColor: '#555' }, tabBarStyle: { backgroundColor: '#555' } }}>
      <Tabs.Screen name="index" path="/" options={{ title: 'Home', icon: 'home', label: 'Home' }} />
      <Tabs.Screen name="about" path="/about" options={{ title: 'About', icon: 'info', label: 'About' }} />
    </Tabs>
  )
}
```

## API

### FileRouter

```tsx
<FileRouter
  routes={RouteObject[]}
  basename="/"
  transitionConfig={{ enabled?: boolean; direction?: 'left' | 'right'; mode?: 'stack' | 'scroll' }}
/>
```

Renders routes from the generated route tree. Pass `routes` from `tamer-router/generated-routes` or your custom output. `transitionConfig` configures native transition animations.

### Stack

Stack layout with AppBar and Content. No TabBar.

```tsx
<Stack titleForPath={(pathname) => string} screenOptions={{ headerStyle?, headerShown? }}>
  <Stack.Screen name="index" path="/" options={{ title?, headerShown? }} />
  <Stack.Screen name="detail" path="/detail" options={{ title? }} />
</Stack>
```

### Tabs

Tabs layout with AppBar, Content, and TabBar.

```tsx
<Tabs titleForPath={(pathname) => string} screenOptions={{ headerStyle?, tabBarStyle?, contentStyle?, iconColor? }}>
  <Tabs.Screen name="index" path="/" options={{ title?, icon?, label?, set? }} />
</Tabs.Screen>
```

`TabsScreenOptions`: `title`, `headerShown`, `icon`, `label`, `set` (icon set).

### useScreenOptions(options)

Call inside a screen to set title/header. Options merged with `Stack.Screen` / `Tabs.Screen`.

### Hardware back: `useBackHandler` / `usePreventBack`

Intercept the system back event **before** the router pops the stack. Handlers are stacked; the **most recently registered** enabled handler runs first.

**You do not need Stack, Tabs, or the rest of the navigation API** (`useTamerNavigate`, generated routes, etc.) to use these hooks. You only need your UI to render under **`FileRouter`** so the back-handler context exists—that can be a **minimal** setup (e.g. one route, one screen, no `Stack` / `Tabs` components). If you skip **`FileRouter`** entirely, the hooks do nothing; you can still listen to **`tamer-router:back`** on **`GlobalEventEmitter`** yourself and call **`NativeModules.TamerRouterNativeModule.didHandleBack(...)`** from custom code.

| Export | Description |
|--------|-------------|
| `useBackHandler(handler, enabled?)` | `handler` returns **`true`** if the event was consumed (router **does not** navigate back), **`false`** to allow default back behavior. Use `'background only'` in the handler body per ReactLynx rules. |
| `usePreventBack(enabled?)` | While `enabled` is true, back is fully consumed (router does not pop). Same as `useBackHandler(() => enabled, enabled)`. |

```tsx
import { useBackHandler, usePreventBack } from '@tamer4lynx/tamer-router'

// Close a modal on back instead of leaving the screen
useBackHandler(() => {
  if (modalOpen) {
    setModalOpen(false)
    return true
  }
  return false
}, modalOpen)

// Block back until the user saves (or while a sheet is open)
usePreventBack(unsavedChanges)
```

Native ↔ JS: after handlers run, the router notifies the host via **`didHandleBack`** so Android (e.g. transition snapshot) and iOS stay in sync.

### useTamerRouter() / useTamerNavigate()

Returns:

| Method | Signature | Description |
|--------|------------|-------------|
| `push` | `(route: string, options?: TransitionOptions) => void` | Push route |
| `replace` | `(route: string, options?: TransitionOptions) => void` | Replace current |
| `back` / `pop` | `(options?: TransitionOptions) => void` | Go back |
| `canGoBack` | `() => boolean` | Whether stack has previous entry |

`TransitionOptions`: `{ mode?: 'stack' \| 'scroll'; direction?: 'left' \| 'right' }`

For tab switching without stack push, use `replace(route, { tab: true })` via `AppShellRouterContext` (used by TabBar).

### Outlet / Slot

Re-exported from react-router. `Slot` is an alias for `Outlet`.

### tamerRouterPlugin

Rsbuild plugin options: `root`, `output`, `srcAlias?`, `layoutFilename?` (default `_layout.tsx`). Generates route file and watches for changes.
