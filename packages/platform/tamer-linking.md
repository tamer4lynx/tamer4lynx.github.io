# tamer-linking

Deep linking and URL handling for Lynx. Requires `LinkingModule`.

## Overview

- **createURL** — Create app URL with scheme, path, query params
- **getInitialURL** — URL that launched the app
- **addEventListener** / **removeEventListener** — Listen for `url` events (`tamer-linking:url`)

## Installation

```bash
t4l add tamer-linking
```

Run **`t4l link`** after installing.

## API

### createURL(path?, options?)

```ts
createURL(path = '', { scheme?, path?, queryParams? }): string
```

Default scheme: `tamerdevapp`. Uses `LinkingModule.createURL` when available.

### getInitialURL()

Returns `Promise<string | null>`.

### addEventListener(type: 'url', listener: URLListener)

Returns `{ remove: () => void }`. Listener receives `{ url: string }`.

### removeEventListener(type: 'url', listener)

Removes the subscription.
