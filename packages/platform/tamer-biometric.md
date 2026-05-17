# tamer-biometric

Biometric authentication for Lynx. Requires `BiometricModule`.

## Overview

- **hasHardwareAsync** — Device has biometric hardware
- **isEnrolledAsync** — User has enrolled biometrics
- **authenticateAsync** — Prompt for biometric auth
- **supportedAuthenticationTypesAsync** — Returns `AuthenticationType[]`

## Installation

```bash
t4l add tamer-biometric
```

Run **`t4l link`** after installing.

## API

### hasHardwareAsync()

Returns `Promise<boolean>`.

### isEnrolledAsync()

Returns `Promise<boolean>`.

### authenticateAsync(options?)

```ts
interface AuthenticateOptions {
  promptMessage?: string
  cancelLabel?: string
  disableDeviceFallback?: boolean
}
```

Returns `Promise<AuthenticateResult>`: `{ success: true }` or `{ success: false; error: string }`.

### supportedAuthenticationTypesAsync()

Returns `Promise<AuthenticationType[]>`.

### AuthenticationType

`FINGERPRINT` (1), `FACIAL_RECOGNITION` (2), `IRIS` (3).
