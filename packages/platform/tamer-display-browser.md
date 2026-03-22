# tamer-display-browser

In-app browser for Lynx. Requires `DisplayBrowserModule`. Used by tamer-auth for OAuth.

## Overview

- **openBrowserAsync** — Open URL in in-app browser
- **openAuthSessionAsync** — Open auth URL, wait for redirect to `redirectUrl`
- **dismissBrowser** — Close the browser

When `openAuthSessionAsync` is not implemented natively, falls back to `openBrowserAsync` + `addEventListener('url')` from tamer-linking.

## Installation

```bash
t4l add tamer-display-browser tamer-linking
```

Run **`t4l link`** after installing.

## API

### openBrowserAsync(url)

Returns `Promise<{ type: string }>`. `type` may be `'opened'` or `'unavailable'`.

### openAuthSessionAsync(url, redirectUrl)

Returns `Promise<AuthSessionResult>`:

```ts
type AuthSessionResult =
  | { type: 'success'; url: string }
  | { type: 'cancel' }
  | { type: 'dismiss' }
```

### dismissBrowser()

Closes the browser. Callback-based; no return value.
