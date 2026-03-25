# tamer-insets

Safe area insets and keyboard state for Lynx. Requires `TamerInsetsModule` native module.

## Overview

- **useInsets()** — Returns `{ top, right, bottom, left, raw }` (safe area insets in px)
- **useKeyboard()** — Returns `{ visible, height, duration, raw }` (keyboard state; `duration` is animation ms for transitions)

Listens to `tamer-insets:change` and `tamer-insets:keyboard` (Android) and `keyboardstatuschanged` (iOS). Falls back to `TamerInsetsModule.getInsets` / `getKeyboard` on mount.

## Installation

```bash
t4l add tamer-insets
```

Run **`t4l link`** after installing.

## API

### useInsets()

Returns `InsetsWithRaw`:

```ts
interface InsetsWithRaw extends Insets {
  top: number
  right: number
  bottom: number
  left: number
  raw: Insets
}
```

### useKeyboard()

Returns `KeyboardStateWithRaw`:

```ts
interface KeyboardStateWithRaw extends KeyboardState {
  visible: boolean
  height: number
  duration: number
  raw: KeyboardState
}
```
