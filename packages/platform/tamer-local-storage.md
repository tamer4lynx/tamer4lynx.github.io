# tamer-local-storage

Web-standard `localStorage` API for Lynx: persistent key-value storage via **Android SharedPreferences** and **iOS UserDefaults**.

## Installation

```bash
t4l add tamer-local-storage
```

Run **`t4l link`** after installing. This package is **not** part of `t4l add-core`; add it explicitly if you need it.

## Usage

**Named import:**

```tsx
import { localStorage } from '@tamer4lynx/tamer-local-storage'

localStorage.setItem('username', 'john_doe')
const username = localStorage.getItem('username')
localStorage.removeItem('username')
localStorage.clear()
```

**Side-effect / global** (optional): `import '@tamer4lynx/tamer-local-storage'` or `import '@tamer4lynx/tamer-local-storage/global'` to attach `globalThis.localStorage` when absent.

## API

Implements the [Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage) surface: `getItem`, `setItem`, `removeItem`, `clear`. Key enumeration (`key`, `length`) is limited on native backends—see the package README on GitHub for details.

## Native

Uses **`lynx.ext.json`** for autolinking. After install, run **`t4l link`**.

## See also

- [Packages overview](/packages/) — other `@tamer4lynx` modules
