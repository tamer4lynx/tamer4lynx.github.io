# tamer-secure-store

Secure key-value storage for Lynx. Uses native keychain/Keystore. Requires `SecureStoreModule`.

## Overview

- **getItemAsync** / **setItemAsync** / **deleteItemAsync** — Async key-value API
- **getItem** / **setItem** — Sync variants (if supported)
- **canUseBiometricAuthentication** — Check if biometric can protect items
- **isAvailableAsync** — Check if secure store is available
- **KeychainAccessibilityConstant** — When items are accessible

## Installation

```bash
t4l add tamer-secure-store
```

Run **`t4l link`** after installing.

## API

### getItemAsync(key, options?)

Returns `Promise<string | null>`. Keys must match `/^[\w.-]+$/`.

### setItemAsync(key, value, options?)

Returns `Promise<void>`. Value must be string.

### deleteItemAsync(key, options?)

Returns `Promise<void>`.

### SecureStoreOptions

```ts
interface SecureStoreOptions {
  keychainService?: string
  requireAuthentication?: boolean
  authenticationPrompt?: string
  keychainAccessible?: KeychainAccessibilityConstant
  accessGroup?: string
}
```

### KeychainAccessibilityConstant

`AFTER_FIRST_UNLOCK`, `AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY`, `ALWAYS`, `WHEN_PASSCODE_SET_THIS_DEVICE_ONLY`, `ALWAYS_THIS_DEVICE_ONLY`, `WHEN_UNLOCKED`, `WHEN_UNLOCKED_THIS_DEVICE_ONLY`.
