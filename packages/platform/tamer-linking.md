# tamer-linking

Deep linking and external URL handling for Lynx. Requires the native `LinkingModule`.

## Install

```bash
t4l add tamer-linking
```

Run **`t4l link`** after installing.

## Usage

### Open an external URL

```ts
import { openURL } from '@tamer4lynx/tamer-linking'

await openURL('https://example.com')
await openURL('mailto:hello@example.com')
await openURL('tamerdevapp://about')
```

### Build an app URL

```ts
import { createURL } from '@tamer4lynx/tamer-linking'

createURL('settings/profile', {
  scheme: 'myapp',
  queryParams: { tab: 'security' },
})
// → 'myapp://settings/profile?tab=security'
```

### Listen for incoming URLs

```ts
import { addEventListener } from '@tamer4lynx/tamer-linking'

useEffect(() => {
  const sub = addEventListener('url', ({ url }) => {
    // route to the right screen based on url
  })
  return () => sub.remove()
}, [])
```

### Read the URL the app launched with

```ts
import { getInitialURL } from '@tamer4lynx/tamer-linking'

const initial = await getInitialURL()
if (initial) navigate(initial)
```

## API

### `openURL(url)`

```ts
openURL(url: string): Promise<boolean>
```

Hands the URL off to the system. Resolves `true` when the OS reports a successful open, `false` otherwise (no handler, malformed URL, native bridge unavailable). Works for `https://`, `mailto:`, `tel:`, `sms:`, App Store / Play Store URLs, and any registered custom scheme.

### `createURL(path?, options?)`

```ts
createURL(path = '', { scheme?, path?, queryParams? }): string
```

Default scheme: `tamerdevapp`. Uses `LinkingModule.createURL` when available.

### `getInitialURL()`

Returns `Promise<string | null>`.

### `addEventListener(type: 'url', listener)`

Returns `{ remove: () => void }`. Listener receives `{ url: string }`.

### `removeEventListener(type: 'url', listener)`

Removes the subscription.

---

## How it works

- **iOS:** `openURL` calls `UIApplication.shared.open(url, options: [:])` on the main thread. `addEventListener('url', …)` listens for `application(_:open:options:)` and `userActivity(.continueUserActivity)` callbacks dispatched by the host.
- **Android:** `openURL` dispatches `Intent.ACTION_VIEW` with `FLAG_ACTIVITY_NEW_TASK`. URL events come from the host's `onCreate` / `onNewIntent` handlers.
- The simulator without Mail.app installed resolves `false` for `mailto:` — that's expected. Real devices and any sim with Mail / Safari installed work normally.
- `createURL` is a pure helper that prefers the native bridge for canonical scheme handling, but falls back to JS string assembly when the bridge isn't available.
