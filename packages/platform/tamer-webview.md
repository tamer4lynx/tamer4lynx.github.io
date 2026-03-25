# tamer-webview

Embedded in-app WebView for Lynx: **iOS** `WKWebView`, **Android** `android.webkit.WebView`. Registers the native custom element **`<webview>`** with props such as `uri`, `html`, and `baseUrl` (no JSON blob on the JS side). There is **no React wrapper component**—you use **`<webview>`** in JSX.

Behavior and API are informed by [react-native-webview](https://github.com/react-native-webview/react-native-webview) (see `reference/react-native-webview` in this repo). Pages can post messages to the app using the same bridge name as RN: **`ReactNativeWebView`** (`window.ReactNativeWebView.postMessage(...)`).

## Installation

```bash
t4l add tamer-webview
```

The CLI resolves the **highest published semver** on npm. Then run **`t4l link`** so native projects pick up `lynx.ext.json`, CocoaPods, and Gradle. **`t4l link`** also refreshes **`.tamer/tamer-components.d.ts`** and your **`tsconfig.json` `include`** when **`syncTamerComponentTypes`** is enabled (default in **`tamer.config.json`**).

## Requirements

- Native host must register the element (autolink does this for Tamer dev app and embeddable).
- **ATS (iOS)** / **cleartext (Android)**: loading non-HTTPS URLs may require app-side configuration (`NSAppTransportSecurity`, `usesCleartextTraffic`, etc.).

## TypeScript

Ambient typings live in **`dist/webview-jsx.d.ts`** (declared in **`package.json`** `exports["./webview-jsx"]` and **`tamer.ambientTypeExports`**). They augment **`@lynx-js/types`** so **`<webview>`** is typed.

**Typings:** **`t4l init`** / **`t4l link`** generate **`.tamer/tamer-components.d.ts`** with **`path`** references into **`node_modules/.../dist/webview-jsx.d.ts`** and update **`tsconfig.json` `include`** when **`syncTamerComponentTypes`** is enabled (default). You do **not** need `import '@tamer4lynx/tamer-webview'` for types alone—that file is enough. If the Lynx app uses **`src/tsconfig.json`**, include **`../.tamer/tamer-components.d.ts`** there as well—see [Ambient types (CLI)](/packages/tooling/tamer-ambient-types).

Import the package only when you use JS exports (imperative helpers):

```tsx
import { useWebViewRef, callWebViewMethod } from '@tamer4lynx/tamer-webview'
```

## Usage

```tsx
<webview
  uri="https://example.com"
  injectedJavaScript="window.__tamer = 1;"
  bindload={(e) => console.log(e.detail.url, e.detail.title)}
  binderror={(e) => console.log(e.detail)}
  bindmessage={(e) => console.log(e.detail.data)}
/>
```

```tsx
<webview html="<p>Hello</p>" baseUrl="https://example.org/" />
```

If **`uri`** is non-empty, it is loaded first. Otherwise **`html`** + optional **`baseUrl`** are used.

## Props (`WebViewProps`)

These match **`packages/tamer-webview/src/types.ts`** (ambient **`<webview>`** uses the same shape).

| Prop | Type | Description |
|------|------|-------------|
| `uri` | `string` | URL to load |
| `html` | `string` | Inline HTML string |
| `baseUrl` | `string` | Base URL when loading `html` |
| `injectedJavaScript` | `string` | Script evaluated after load |
| `injectedJavaScriptBeforeContentLoaded` | `string` | Script evaluated before the page’s own scripts run |
| `javaScriptEnabled` | `boolean` | Enable/disable JavaScript in the page |
| `messagingEnabled` | `boolean` | When enabled, exposes `window.ReactNativeWebView.postMessage` to the page |
| `userAgent` | `string` | Override the WebView user agent |
| `style` | `string \| CSSProperties` | Lynx styles |
| `className` | `string` | CSS class name |
| `id` | `string` | Element id (e.g. for selector queries) |
| `bindload` | handler | See [Events](#events) |
| `binderror` | handler | See [Events](#events) |
| `bindmessage` | handler | See [Events](#events) |

Native defaults (not encoded in TypeScript): **`javaScriptEnabled`** and **`messagingEnabled`** default to **`true`** on iOS/Android; **`userAgent`** falls back to a Chrome Mobile–style UA when unset.

## Events

Handlers use Lynx **`BaseEvent`**: **`e.detail`** carries the payload (not React-style).

| Bind attribute | `e.detail` type |
|----------------|-------------------|
| `bindload` | `{ url: string; title: string; loading: boolean; canGoBack: boolean; canGoForward: boolean }` |
| `binderror` | `{ domain?: string; code?: number; description?: string }` |
| `bindmessage` | `{ data: string }` (from `window.ReactNativeWebView.postMessage`) |

### Imperative APIs (`invoke` / `WebViewRef`) {#imperative-apis}

Use [`NodesRef.invoke`](https://lynxjs.org/api/lynx-api/nodes-ref/nodes-ref-invoke.html) with **`callWebViewMethod`**, or **`useWebViewRef()`** — the ref’s imperative surface is **`WebViewRef`** in **`packages/tamer-webview/src/index.tsx`**.

| Method | Params | Description |
|--------|--------|-------------|
| `reload` | — | Reload the current page |
| `goBack` | — | Go back in history |
| `goForward` | — | Go forward in history |
| `stopLoading` | — | Stop the current load |
| `loadUrl` | `{ url: string }` | Load a URL |
| `injectJavaScript` | `{ script: string }` | Run JS in the page |
| `postMessage` | `{ data: string }` | Dispatch a `message` event on `window` in the page |

## See also

- [Ambient types (CLI)](/packages/tooling/tamer-ambient-types) — `.tamer/tamer-components.d.ts` and `tsconfig` wiring
- [Custom Element (Lynx)](https://lynxjs.org/guide/custom-native-component.html)
- [NodesRef `invoke`](https://lynxjs.org/api/lynx-api/nodes-ref/nodes-ref-invoke.html)
