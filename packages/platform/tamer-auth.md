# tamer-auth

OAuth 2.0 / PKCE auth flow for Lynx. Depends on tamer-linking and tamer-display-browser.

## Overview

- **AuthRequest** — Build auth URL, prompt user, parse callback
- **makeRedirectUri** — Create redirect URI for OAuth
- **exchangeCodeAsync** — Exchange authorization code for tokens
- **TokenResponse** — Token storage and refresh

Uses PKCE by default. Opens auth URL via `openAuthSessionAsync` (tamer-display-browser), parses redirect for `code` and `state`.

## Installation

```bash
t4l add tamer-auth tamer-linking tamer-display-browser
```

Run **`t4l link`** after installing.

## API

### AuthRequest

```ts
new AuthRequest(config: AuthRequestConfig)
```

`AuthRequestConfig`: `clientId`, `redirectUri`, `scopes?`, `responseType?`, `usePKCE?`, `state?`, `extraParams?`

- `makeAuthUrlAsync(discovery: AuthDiscoveryDocument): Promise<string>` — Build auth URL
- `promptAsync(discovery): Promise<AuthSessionResult>` — Open browser, wait for redirect
- `parseReturnUrl(url)` — Parse callback URL for code/state

### makeRedirectUri(options?)

```ts
makeRedirectUri({ scheme?, path? }): string
```

Default scheme: `tamerdevapp`. Default path: `auth/callback`.

### exchangeCodeAsync

Exchange authorization code for tokens. See `AuthRequest` and `TokenResponse` types.

### TokenResponse

Token storage. See source for methods.
