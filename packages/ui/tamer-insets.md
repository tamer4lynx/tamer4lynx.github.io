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

## Platform notes

### iOS (v0.0.4+)

Call **`TamerInsetsModule.attachHostView(lynxView)`** after adding your LynxView to the hierarchy so safe-area and keyboard overlap match the Lynx surface. Call **`attachHostView(nil)`** before tearing down. Templates in tamer-host / tamer-dev-client do this automatically.

The keyboard visibility threshold was relaxed from `> 0.5pt` to `> 0pt` in v0.0.4, fixing detection on iOS 26.2 where keyboard height above safe area is minimal.

### Android

After keyboard height updates, the module reschedules a few delayed inset re-reads to handle custom ROMs that report IME size incorrectly on first frames.
