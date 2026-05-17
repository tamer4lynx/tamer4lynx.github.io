# Platform packages

Networking polyfills, storage, auth, linking, haptics, and embedded web content.

| Package | Role |
|---------|------|
| [tamer-transports](/packages/platform/tamer-transports) | Fetch, WebSocket, EventSource polyfills |
| [tamer-local-storage](/packages/platform/tamer-local-storage) | Web `localStorage` on native |
| [jiggle](/packages/platform/jiggle) | Vibration / haptics |
| [tamer-auth](/packages/platform/tamer-auth) | OAuth 2.0 / PKCE |
| [tamer-secure-store](/packages/platform/tamer-secure-store) | Secure key-value storage |
| [tamer-biometric](/packages/platform/tamer-biometric) | Biometric authentication |
| [tamer-linking](/packages/platform/tamer-linking) | Deep links and URL handling |
| [tamer-display-browser](/packages/platform/tamer-display-browser) | In-app browser (e.g. OAuth) |
| [tamer-webview](/packages/platform/tamer-webview) | Native `<webview>` element |

Add only what you need with **`t4l add <name>`**. **`t4l add-core`** includes **tamer-transports** but not auth, webview, or local storage.
