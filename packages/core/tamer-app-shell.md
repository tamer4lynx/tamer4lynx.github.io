# tamer-app-shell

App chrome (AppBar / TabBar / Screen / SafeArea) plus a Material 3 component set (Button, Card, FAB, NavigationDrawer, NavigationRail, …) for Lynx. Used by tamer-router `Stack` / `Tab` layouts and standalone in any Lynx page.

## Install

```bash
t4l add-core
# or: t4l add tamer-app-shell tamer-icons tamer-insets tamer-screen
```

Run **`t4l link`** after installing.

## Usage

Each export drops in directly — most pages need only a few. Imports first:

```tsx
import {
  AppBar,
  TabBar,
  Screen,
  SafeArea,
  Button,
  ButtonGroup,
  Card,
  Fab,
  ExtendedFab,
  FabMenu,
  FloatingFabContainer,
  NavigationDrawer,
  NavigationRail,
  ScreenScopedOverlay,
  SCREEN_OVERLAY_LEVEL_DRAWER,
  SCREEN_OVERLAY_LEVEL_FLOATING,
  useM3ThemeTokens,
} from '@tamer4lynx/tamer-app-shell'
```

### Button

```tsx
<Button variant="outlined" icon="add" size="md" onTap={...}>New</Button>
```

`variant`: `'filled'` (default) | `'outlined'` | `'text'` | `'elevated'` | `'tonal'`  
`size`: `'xs'` | `'sm'` | `'md'` (default) | `'lg'` | `'xl'`  
`shape`: `'round'` (default) | `'square'`

### ButtonGroup

```tsx
<ButtonGroup
  items={[
    { id: 'left',  label: 'Left'   },
    { id: 'mid',   label: 'Middle' },
    { id: 'right', label: 'Right'  },
  ]}
  selectedId={selected}
  onSelect={setSelected}
/>
```

### Card

```tsx
<Card variant="elevated">{...children}</Card>
```

`variant`: `'elevated'` (default) | `'filled'` | `'outlined'`

### Fab

```tsx
<Fab icon="add" size="regular" onTap={...} />
```

`size`: `'small'` | `'regular'` (default) | `'large'`

### ExtendedFab

```tsx
<ExtendedFab icon="edit" onTap={...}>Compose</ExtendedFab>
```

### FabMenu

```tsx
<FabMenu
  items={[
    { id: 'a', icon: 'image', label: 'Photo', onTap: () => {} },
    { id: 'b', icon: 'mic',   label: 'Voice', onTap: () => {} },
  ]}
/>
```

### FloatingFabContainer

```tsx
<FloatingFabContainer>
  <Fab icon="add" onTap={...} />
</FloatingFabContainer>
```

### NavigationDrawer

```tsx
<NavigationDrawer
  open={open}
  onDismiss={() => setOpen(false)}
  sections={[
    {
      header: 'Mail',
      items: [
        { id: 'inbox', icon: 'inbox', label: 'Inbox' },
        { id: 'sent',  icon: 'send',  label: 'Sent'  },
      ],
    },
  ]}
  selectedId={selected}
  onSelect={setSelected}
/>
```

### NavigationRail

```tsx
<NavigationRail
  items={[
    { id: 'home',  icon: 'home' },
    { id: 'fav',   icon: 'star' },
    { id: 'about', icon: 'info' },
  ]}
  selectedId={selected}
  onSelect={setSelected}
/>
```

### ScreenScopedOverlay

```tsx
<ScreenScopedOverlay level={SCREEN_OVERLAY_LEVEL_DRAWER} visible={open}>
  ...overlay content...
</ScreenScopedOverlay>
```

`level`: `SCREEN_OVERLAY_LEVEL_FLOATING` (10) for menus/FABs, `SCREEN_OVERLAY_LEVEL_DRAWER` (20) for drawers.

### Theme tokens

```tsx
const t = useM3ThemeTokens()

<view style={{ backgroundColor: t.surface, borderColor: t.outlineVariant }}>
  <text style={{ color: t.onSurface }}>Hi</text>
</view>
```

The token set tracks the active light / dark palette from [tamer-system-ui](/packages/ui/tamer-system-ui), so theme switches propagate without per-component plumbing.

The `packages/example` app at `pages/m3/` exercises every component above — see [Example Anatomy](/guide/example-anatomy).

## API

### App chrome

#### `<AppBar>`

```tsx
<AppBar
  title?: string
  barHeight?: number
  leftAction?: AppBarAction | false
  rightActions?: AppBarAction[]
  foregroundColor?: string
  actionColor?: string
  style?: ViewProps['style']
/>
```

`AppBarAction`: `{ icon: string; set?: IconSet; onTap: () => void }`. If `leftAction` is undefined and `canGoBack()`, shows the default back button.

#### `<TabBar>`

```tsx
<TabBar
  tabs={TabItem[]}
  iconColor?: { active?: string; inactive?: string }
  style?: ViewProps['style']
/>
```

`TabItem`: `{ icon: string; set?: IconSet; label?: string; path?: string; onTap?: () => void }`. Uses `AppShellRouterContext.replace(path, { tab: true })` when `path` is set.

#### `<Content>`

Scrollable view. Accepts `children` and optional `style`.

#### `<Screen>` / `<SafeArea>`

Re-exported from [tamer-screen](/packages/ui/tamer-screen). `Screen` is a full-screen flex column. `SafeArea` accepts `edges?: ('top'|'right'|'bottom'|'left')[]` plus `children`.

#### `<AppShellProvider>` / `useAppShellContext()`

```tsx
<AppShellProvider showAppBar showTabBar barHeight?>{children}</AppShellProvider>
```

`useAppShellContext()` returns `{ showAppBar, showTabBar, barHeight }`.

#### `useAppShellRouter()`

Returns `AppShellRouterContextValue | null`: `back`, `canGoBack`, `replace(route, options?)`. Provided by `FileRouter`; usable directly with manual coordinators.

### Material 3 components

#### `<Button>`

```tsx
<Button
  variant?: 'filled' | 'outlined' | 'text' | 'elevated' | 'tonal'
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  shape?: 'round' | 'square'
  icon?: string
  iconSet?: IconSet
  onTap?: () => void
  disabled?: boolean
>
  Label
</Button>
```

#### `<ButtonGroup>`

`ButtonGroupProps` accepts `items: ButtonGroupItem[]`, `selectedId`, `onSelect`. Connected button row.

#### `<Card variant>`

`'elevated' | 'filled' | 'outlined'`.

#### `<Fab>` / `<ExtendedFab>` / `<FabMenu>`

`FabSize`: `'small' | 'regular' | 'large'`. `FabMenu` takes `FabMenuItem[]` (id, icon, label, onTap).

#### `<FloatingFabContainer>` / `useFloatingFabOffsets()`

Wraps a FAB at the screen edge accounting for tab bar + insets. The hook returns `{ bottom, right, … }` in px if you want the math without the wrapper.

#### `<ScreenScopedOverlay>`

Mounts an overlay (sheet, drawer, dialog) above the current screen — sits below pushed routes. Use `level={SCREEN_OVERLAY_LEVEL_FLOATING}` (10) for menus / FABs and `level={SCREEN_OVERLAY_LEVEL_DRAWER}` (20) for drawers.

#### `<NavigationDrawer>`

Side drawer; takes `DrawerSection[]` of `DrawerItem`s, `open`, `onDismiss`, `selectedId`, `onSelect`.

#### `<NavigationRail>`

Vertical rail; takes `NavRailItem[]`, `selectedId`, `onSelect`.

#### `useM3ThemeTokens()`

Returns the M3 token set (`primary`, `onPrimary`, `surface`, `surfaceContainer*`, `outline`, `outlineVariant`, …) derived from the active palette in [tamer-system-ui](/packages/ui/tamer-system-ui).

#### `px(...values)`

Helper for converting a list of numbers to a px string.

---

## How it works

- **App chrome** is a thin layer over plain Lynx `<view>` / `<text>` — no native module. `AppBar` / `TabBar` read insets from [tamer-insets](/packages/ui/tamer-insets) and palette from [tamer-system-ui](/packages/ui/tamer-system-ui). `Screen` / `SafeArea` come from [tamer-screen](/packages/ui/tamer-screen).
- **`AppShellRouterContext`** is populated by `FileRouter` (or your own coordinator). Components that show a back button (`AppBar`, `NavigationDrawer` headers) consume it via `useAppShellRouter()`.
- **M3 components** consume `useM3ThemeTokens()` directly, so any theme change propagates without re-rendering the whole tree manually.
- **`ScreenScopedOverlay`** mounts into the active screen's overlay layer, not document root — pushed routes hide existing overlays automatically and overlays don't bleed across stack entries.
- **`FloatingFabContainer`** subscribes to `useInsets()` + a constant tab-bar visual height (80px); `useFloatingFabOffsets()` exposes the same math.
