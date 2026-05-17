# tamer-router

File-based routing for Lynx with React Router 6 + TamerNav native stack coordination. Covers file-based route generation, hardware back handling, and cross-spoke state synchronization.

## Overview

- **File-based routing:** Rsbuild plugin scans `pages/` → generates route tree. Conventions: `index` → index route, `[param]` → dynamic segment, `_layout.tsx` → layout wrapper.
- **Native stack:** `FileRouter` pushes spoke LynxViews via `TamerNav` from `@tamer4lynx/tamer-navigation`. Coordinator manages route stack.
- **Layout components:** `Stack` / `Tabs` with AppBar, TabBar, Content (via tamer-app-shell).
- **State bridging:** `providerConnector` prop syncs React Context / hooks across spoke boundaries. **Required if your app uses any React providers (Zustand, Redux, TanStack Query, i18n, theme, etc.)** — each spoke gets a fresh JS context and won't inherit the coordinator's provider state without this. See **Cross-spoke state** below.
- **Hardware back:** `BackHandlerProvider` + `useBackHandler` intercept Android back / iOS gesture.
- **Manual coordinator:** Don't need `FileRouter`? Use `TamerNav` directly from `@tamer4lynx/tamer-navigation` with `BackHandlerProvider` for back handling. The example app's [`src/example_stack.tsx`](https://github.com/tamer4lynx/tamer4lynx/blob/main/packages/example/src/example_stack.tsx) shows this pattern — a hand-rolled coordinator that calls `TamerNav.push` / `TamerNav.pop` without any file-based routing. `tamer-router` is a higher-level solution built on top of the same primitives.

:::tip Refactored recently
`tamer-router` has gone through a significant internal refactor. If you are one of the handful of people already using it — you know who you are — the public API is the same but the internals are cleaner. If it breaks, please file an issue.
:::

## Installation

```bash
t4l add tamer-router
```

Peers: **`react-router@^6`**. Run **`t4l link`** after install.

## Setup

### 1. Rsbuild config

Use **tamer-plugin** (applies default tamer-router config automatically):

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

Or configure **tamerRouterPlugin** directly:

```ts
import { tamerRouterPlugin } from '@tamer4lynx/tamer-router'

export default {
  plugins: [
    tamerRouterPlugin({
      root: './src/pages',
      srcAlias: '@/',
      layoutFilename: '_layout.tsx',
    }),
  ],
}
```

### 2. Entry point

```tsx
import { root } from '@lynx-js/react'
import { FileRouter } from '@tamer4lynx/tamer-router'

root.render(<FileRouter />)
```

Optional: pass `providerConnector` to bridge state across spokes (see **Cross-spoke state** below).

### 3. Layout (example)

`src/pages/_layout.tsx`:

```tsx
import { Stack } from '@tamer4lynx/tamer-router'
import { useSystemUI } from '@tamer4lynx/tamer-system-ui'

export default function Layout() {
  const { setStatusBar } = useSystemUI()

  useEffect(() => {
    setStatusBar({ color: '#fff', style: 'light' })
  }, [])

  return (
    <Stack>
      <Stack.Screen name="index" path="/" options={{ title: 'Home' }} />
      <Stack.Screen name="detail" path="/detail/:id" options={{ title: 'Detail' }} />
    </Stack>
  )
}
```

## API

### `<FileRouter>`

Auto-generates routes from `pages/` directory and manages coordinator/spoke pushing via TamerNav.

```tsx
<FileRouter
  children?: ReactNode
  lazyRoutes?: boolean
  providerConnector?: TamerProviderConnector[]
  basename?: string
  knownPaths?: string[]
  rootBackgroundColor?: string
  exitOnRootHardwareBack?: boolean
/>
```

| Prop | Type | Description |
|------|------|-------------|
| `providerConnector` | `TamerStateSync[]` | State syncs to bridge across spokes (optional). See **Cross-spoke state**. |
| `exitOnRootHardwareBack` | `boolean` | Exit app if back pressed at root (default: false) |
| Other props | | React Router config (`basename`, etc.) |

### Cross-spoke state: `providerConnector`

Each spoke LynxView gets a fresh JS context. Module-level singletons (Zustand, Redux) re-evaluate per spoke. React Context set on the coordinator doesn't survive into spokes. **If your app uses any React provider — state management, theming, i18n, data fetching — you need `providerConnector` to carry that state across screen pushes.**

Pass `providerConnector` to bridge state explicitly:

```tsx
import { FileRouter, createZustandSync } from '@tamer4lynx/tamer-router'
import { myStore } from './store'

const mySync = createZustandSync('myStore', myStore)

<FileRouter providerConnector={[mySync]} />
```

On push, `FileRouter` serializes all syncs to JSON and hydrates spokes. On mutation, spokes re-serialize back via `TamerNav.update`. Coordinators listen to `tamer-nav:dispatch` events.

### Built-in connectors

`tamer-router` ships purpose-built connectors for the libraries the example app exercises. Each one is a thin wrapper around `createTamerStateSync` that knows how to serialize / hydrate that library's store and (for Redux-style libs) how to forward dispatched actions across spokes.

| Connector | Library | What it carries across spokes | Spoke → coordinator action forwarding |
|-----------|---------|-------------------------------|---------------------------------------|
| `createZustandSync(key, store)` | [Zustand](https://github.com/pmndrs/zustand) | Whole store snapshot via `store.getState()` / `store.setState(parsed, true)` | No (mutate via `setState` on the spoke; coordinator hydrates back) |
| `createReduxSync(key, store)` | [Redux](https://redux.js.org/) | `store.getState()` snapshot; rehydrates with a synthetic `@@TAMER_HYDRATE` action | Yes — `store.dispatch` on a spoke is forwarded as a `tamer-nav:dispatch` event and replayed on the coordinator |
| `createTanstackQuerySync(key, queryClient)` | [TanStack Query v5](https://tanstack.com/query) | Serialized query cache (`dehydrate` / `hydrate` from `@tanstack/react-query`) | Optional — exposes a `send` for invalidation events |
| `createApolloSync(key, client)` | [Apollo Client](https://www.apollographql.com/) | `client.cache.extract()` / `client.cache.restore()` | No |
| `createSwrSync(key, cache)` | [SWR](https://swr.vercel.app/) | Tracked SWR cache (use `createTrackedSwrCache()` from this package as the SWR provider's `provider`) | No |
| `createJotaiSync(key, store)` | [Jotai](https://jotai.org/) | Atom snapshot for atoms registered with the sync | No |
| `createRecoilSync(key, ...)` | [Recoil](https://recoiljs.org/) | Tracked atoms via the Recoil snapshot API | No |
| `createI18nextSync(key, i18n)` | [i18next](https://www.i18next.com/) | Active language + loaded resource bundles | No |
| `createThemeSync(key, themeStore)` | Tamer-internal theme store / generic palette object | Theme state as plain JSON | No |

```tsx
import { FileRouter, createZustandSync, createReduxSync } from '@tamer4lynx/tamer-router'
import { useStore } from './useStore'
import { reduxStore } from './reduxStore'

const connectors = [
  createZustandSync('app', useStore),
  createReduxSync('redux', reduxStore),
]

<FileRouter providerConnector={connectors} />
```

Each connector picks a unique `key` — keep them stable across releases (the JSON aggregate is keyed by it).

### Connector lifecycle

The coordinator owns the `providerConnector` array and runs the aggregate. Per push/pop the flow is:

1. **On `TamerNav.push`** — coordinator calls `serialize()` on each sync, builds `{ [key]: <slice> }`, and passes the aggregate as `stateJson`.
2. **Spoke mounts** — `FileRouter` (mounted on the spoke side as well) reads `readHydratedStateJson()` once, splits the aggregate by key, and calls each sync's `hydrate(json)`. The store now matches the coordinator's snapshot.
3. **On spoke mutation** — when a tracked store updates, the spoke re-serializes and broadcasts via `TamerNav.update({ stateJson })`. Coordinator receives it, re-hydrates its own copy, then **all other open spokes** also re-hydrate. Stores stay coherent across the stack.
4. **For libraries with actions (Redux):** the spoke's `store.dispatch(action)` is also wrapped — the action is sent as a `tamer-nav:dispatch` event and replayed on the coordinator before the next `update` fires.

If `dispatchConnectorMutations` is enabled, the same dispatch round-trip is applied to every connector, not just Redux.

### Writing a connector for an unsupported provider

If your library isn't in the table above, you can still bridge it. Build a `TamerStateSync` directly:

```ts
import { createTamerStateSync } from '@tamer4lynx/tamer-router'
import { myStore } from './myStore'

const mySync = createTamerStateSync('my-store', {
  getState: () => myStore.snapshot(),
  subscribe: (listener) => myStore.on('change', listener),
  hydrate: (json) => myStore.replace(JSON.parse(json)),
  // Optional — implement only if your store has a dispatch / action concept worth replaying
  send: (action) => myStore.dispatch(action),
})

<FileRouter providerConnector={[mySync]} />
```

Contract:

- **`getState()`** — return the JSON-serializable snapshot. If your store holds non-serializable values (functions, class instances, dates), strip / convert them here.
- **`subscribe(listener)`** — call `listener()` whenever state changes. Return an unsubscribe function.
- **`hydrate(json)`** — accept the JSON snapshot and replace local state. **Idempotent** — `tamer-router` may call this multiple times during cross-spoke updates.
- **`send(action)`** — optional. Use only when there's a meaningful action object (Redux, Pinia mutations) you want replayed on every spoke. For pure state-replication libs (Zustand, Apollo cache), leave it off.

`createTamerStateSync` is intentionally tiny — every built-in connector is just a thin shim that knows the library's API for the four bullets above. If you author one for a popular library, please file a PR so it can be added to the built-in list.

### Non-React Lynx bindings (miso-lynx, VueLynx, …)

`tamer-router` is React-only. For Vue or Haskell Lynx bindings, use [`tamer-navigation`](/packages/core/tamer-navigation#using-tamernav-with-non-react-lynx-bindings) directly — that page lays out the equivalent state-hydration patterns for Vue's Pinia, Haskell's effect models, and other bindings.

### `<Stack>`

Stack navigation with AppBar and Content.

```tsx
<Stack screenOptions={{ headerStyle?, headerShown? }}>
  <Stack.Screen name="index" path="/" options={{ title?, headerShown? }} />
  <Stack.Screen name="detail" path="/detail/:id" options={{ title? }} />
</Stack>
```

### `<Tabs>`

Tabs navigation with AppBar, Content, and TabBar.

```tsx
<Tabs screenOptions={{ headerStyle?, tabBarStyle?, contentStyle?, iconColor? }}>
  <Tabs.Screen name="index" path="/" options={{ title?, icon?, label? }} />
  <Tabs.Screen name="settings" path="/settings" options={{ title?, icon?, label? }} />
</Tabs>
```

### `useBackHandler` / `usePreventBack`

Intercept hardware back (Android) / pop gesture (iOS) **before** default handling.

```tsx
import { useBackHandler, usePreventBack } from '@tamer4lynx/tamer-router'

// Close modal instead of leaving screen
useBackHandler(() => {
  if (modalOpen) {
    setModalOpen(false)
    return true  // consumed
  }
  return false   // pass through
})

// Block back while unsaved
usePreventBack(isDirty)
```

Handlers are LIFO. When none return `true`, `FileRouter` pops the stack. In manual coordinator setups, you handle the fallback yourself.

### `useTamerRouter()`

```tsx
const { push, replace, back, canGoBack, navigate } = useTamerRouter()

push('/detail/42')
replace('/home')
back()
```

### Re-exports from React Router

`useLocation`, `useNavigate`, `useParams`, `useLocalSearchParams`, `Outlet` / `Slot`, `Link`
