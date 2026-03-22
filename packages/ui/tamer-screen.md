# tamer-screen

Screen, SafeArea, and AvoidKeyboard components. Re-exports `useInsets` and `useKeyboard` from tamer-insets.

## Overview

- **Screen** — Full-screen flex column container
- **SafeArea** — Applies safe area insets as padding; provides `SafeAreaContext`
- **AvoidKeyboard** — Adjusts layout when keyboard is visible
- **useSafeAreaContext()** — Returns `{ hasTop, hasRight, hasBottom, hasLeft }` when inside SafeArea

## Installation

```bash
t4l add tamer-screen tamer-insets
```

Run **`t4l link`** after installing.

## API

### Screen

```tsx
<Screen style?>{children}</Screen>
```

Flex column, full width/height, minHeight 0.

### SafeArea

```tsx
<SafeArea edges?: ('top'|'right'|'bottom'|'left')[] style?>{children}</SafeArea>
```

Applies `useInsets()` as padding on specified edges. Default `edges` is all four. Provides `SafeAreaContext` to children.

### AvoidKeyboard

```tsx
<AvoidKeyboard behavior?: 'padding'|'position' animate?: boolean style?>{children}</AvoidKeyboard>
```

When keyboard is visible, adds bottom padding (`padding`) or `bottom` offset (`position`). Cancels bottom safe area inset when using padding. `animate={false}` snaps into place instead of animating.

### useSafeAreaContext()

Returns `{ hasTop, hasRight, hasBottom, hasLeft } | null`. Non-null when inside a SafeArea with that edge enabled.
