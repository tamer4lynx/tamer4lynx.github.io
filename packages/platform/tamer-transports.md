# tamer-transports

Fetch, WebSocket, and EventSource polyfills for Lynx. Required for HMR and WebSocket in native Lynx apps.

## Overview

Installs polyfills on import:

- **installFetchPolyfill()** — Replaces Lynx’s stock fetch with a polyfill that attempts to meet the browser-standard fetch API
- **installWebSocketPolyfill()** — WebSocket polyfill that attempts to meet the browser-standard WebSocket API
- **installEventSourcePolyfill()** — EventSource polyfill that attempts to meet the browser-standard EventSource API

Exports `fetch`, `WebSocket`, `EventSource` (native or polyfilled).

**Background thread:** In Lynx, **`fetch`**, **`WebSocket`**, and **`EventSource`** are **background-thread** APIs (same as the host `fetch` in the [runtime docs](https://lynxjs.org/guide/scripting-runtime/index.md)). Do not call them from main-thread-only code. **Best practice:** perform network setup and any state updates from **`useEffect`**, event handlers, or other background-thread code, and add **`'background only'`** as the first statement in that function when the compiler cannot infer it (e.g. when setting React state after a fetch inside a custom hook).

**Example (ReactLynx):** open a WebSocket in an effect, update state from messages, clean up on unmount:

```tsx
import { useEffect, useState } from '@lynx-js/react';
import { WebSocket } from '@tamer4lynx/tamer-transports';

export function App() {
  const [text, setText] = useState('Hello');

  useEffect(() => {
    'background only';
    let released = false;
    const ws = new WebSocket('wss://example.com/socket');

    ws.onmessage = (event) => {
      if (released) return;
      setText(String(event.data));
    };
    ws.onerror = (event) => {
      console.error('ws error', event);
    };

    return () => {
      released = true;
      ws.onmessage = null;
      ws.onerror = null;
      try {
        if (ws.readyState === WebSocket.CONNECTING || ws.readyState === WebSocket.OPEN) {
          ws.close(1000, 'cleanup');
        }
      } catch {
        /* ignore */
      }
    };
  }, []);

  return (
    <view>
      <text>{text}</text>
    </view>
  );
}
```

> **Note:** These polyfills are not fully tested. Report issues on GitHub.

## Installation

```bash
t4l add tamer-transports
```

Run **`t4l link`** after installing. Import early in your app entry (e.g. before any fetch/WebSocket usage):

```ts
import '@tamer4lynx/tamer-transports'
```

## API

The package has no public API beyond the polyfills. Importing it installs the polyfills globally. Use `globalThis.fetch`, `new WebSocket(...)`, and `EventSource` as usual.
