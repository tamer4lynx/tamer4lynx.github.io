# tamer-app-shell

AppBar, TabBar, Content, SafeArea, and Screen components for Lynx. Used by tamer-router Stack and Tabs layouts.

## Overview

- **AppBar** — Title bar with back button, actions
- **TabBar** — Bottom tab bar with icons and labels
- **Content** — Scrollable content area
- **Screen** — Full-screen flex container
- **SafeArea** — Padding for safe area insets
- **AppShellProvider** — Context for showAppBar/showTabBar
- **AppShellRouterContext** — Back/replace for navigation (provided by FileRouter)

## Installation

```bash
t4l add-core
# or: t4l add tamer-app-shell tamer-icons tamer-insets tamer-screen
```

Run **`t4l link`** after installing.

## API

### AppBar

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

`AppBarAction`: `{ icon: string; set?: IconSet; onTap: () => void }`. If `leftAction` is undefined and `canGoBack()`, shows default back button.

### TabBar

```tsx
<TabBar
  tabs={TabItem[]}
  iconColor?: { active?: string; inactive?: string }
  style?: ViewProps['style']
/>
```

`TabItem`: `{ icon: string; set?: IconSet; label?: string; path?: string; onTap?: () => void }`. Uses `AppShellRouterContext.replace(path, { tab: true })` when path is set.

### Content

Scrollable view. Pass `children` and optional `style`.

### Screen

Full-screen flex column. Pass `children` and optional `style`.

### SafeArea

Re-exported from tamer-screen. Pass `edges?: ('top'|'right'|'bottom'|'left')[]` and `children`.

### AppShellProvider

```tsx
<AppShellProvider showAppBar showTabBar barHeight?>{children}</AppShellProvider>
```

### useAppShellContext()

Returns `{ showAppBar, showTabBar, barHeight }`.

### useAppShellRouter()

Returns `AppShellRouterContextValue | null`: `back`, `canGoBack`, `replace(route, options?)`.
