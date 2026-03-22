# tamer-host

Production Lynx host templates for injecting LynxView into existing Android and iOS apps.

## Overview

`tamer-host` provides the minimal native infrastructure needed to run a Lynx bundle in an existing app:

- **Android**: `App.java`, `TemplateProvider.java`, `MainActivity.kt`
- **iOS**: `AppDelegate.swift`, `SceneDelegate.swift`, `ViewController.swift`, `LynxProvider.swift`, `LynxInitProcessor.swift`

## Installation

```bash
t4l add tamer-host
```

Run **`t4l link`** after installing.

## Usage

### Inject into an existing project

After creating an Android or iOS project (or if you already have one), run:

```bash
t4l inject android
t4l inject ios
```

Use `--force` to overwrite existing files:

```bash
t4l inject android --force
```

### Embeddable output (no inject required)

Build a production Lynx bundle plus code snippets you can add to any existing app:

```bash
t4l build android --embeddable
```

Outputs to `embeddable/`: the bundle, Android (Kotlin) and iOS (Swift) snippets, and a README.

### Prerequisites

- `tamer.config.json` with `android.packageName` / `ios.appName` and `ios.bundleId`
- An existing Android project (`android/app/src/main/java/`, `android/app/src/main/kotlin/`) or iOS project (`ios/<AppName>/`)
- Run `t4l link` after injecting to register native modules

### Create flow

`t4l create android` and `t4l create ios` use `tamer-host` templates when the package is installed. If `tamer-host` is not installed, the CLI falls back to inline templates.

## Related

- [tamer-dev-client](/packages/core/tamer-dev-client) — Adds dev launcher (QR scan, HMR, `meta.json` compatibility) on top of the host; iOS embedded flow syncs `LynxInitProcessor` + optional `tamer-host-native-modules.json` via `t4l link ios`
- [Embedding LynxView into Native View](https://lynxjs.org/guide/embed-lynx-to-native) — Lynx guide for embedding LynxView in existing layouts
