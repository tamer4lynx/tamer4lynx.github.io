# @tamer4lynx/jiggle

Vibration/haptic native module for Lynx. Exposes `vibrate(durationMs)` to the Lynx runtime.

## Installation

```bash
t4l add jiggle
```

Run **`t4l link`** after installing.

## API

| Method | Description |
|--------|-------------|
| `vibrate(durationMs)` | Vibrates the device for the provided duration in milliseconds |

**Parameters:** `durationMs` (number) — duration in milliseconds.

**Usage:**

```js
if (global.NativeModules?.Jiggle) {
  NativeModules.Jiggle.vibrate(200)
}
```

## Platform

Uses **lynx.ext.json**. Run `t4l link` after adding to your app.
