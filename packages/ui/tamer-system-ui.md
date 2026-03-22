# tamer-system-ui

Status bar, navigation bar, and theme colors for Lynx. Requires `SystemUIModule` native module.

## Overview

- **setStatusBar** — Set status bar style (light/dark/auto)
- **setNavigationBar** — Set nav bar color and style
- **setRootBackground** — Set root view background
- **getThemeColors** / **getThemeColorsAsync** — Get system theme colors
- **useThemeColors()** — Reactive hook for theme colors; listens to `system-ui:themeChanged`

## Installation

```bash
t4l add tamer-system-ui
```

Run **`t4l link`** after installing.

## API

### useSystemUI()

Returns `{ setStatusBar, setNavigationBar, setRootBackground, getThemeColors, getThemeColorsAsync }`.

### setStatusBar(options)

```ts
interface StatusBarOptions {
  color?: string   // Background color (for 'auto' style)
  style?: 'light' | 'dark' | 'auto'
}
```

When `style` is `'auto'`, derives light/dark from `color` using WCAG luminance.

### setNavigationBar(options)

```ts
interface NavigationBarOptions {
  color: string
  style?: 'light' | 'dark' | 'auto'
}
```

### setRootBackground(options)

```ts
{ color: string }
```

### ThemeColors

```ts
interface ThemeColors {
  primary?: string
  primaryDark?: string
  background?: string
  surface?: string
  surfaceContainer?: string
  onSurface?: string
  isDark?: boolean
}
```

### useThemeColors()

Returns `ThemeColors | null`. Updates on `system-ui:themeChanged` or refetch.
