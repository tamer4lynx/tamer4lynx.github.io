# Getting Started

There are two flows for running a Lynx project against Tamer. Pick one — your selection sticks for the rest of the page. Both flows start by installing the global **`t4l`** CLI from npm.

![](/assets/icon-tamer.png)Tamer App (prebuilt)DevDev Client (build it yourself)The fastest path. Install the **prebuilt Tamer Dev App** on your phone or simulator, run `t4l start` from your project, scan the QR. No Xcode / Android SDK / native build required.
:::tip Use this flow when…
You want to iterate on Lynx UI without touching native code, or you don't have Xcode / Android SDK installed locally.
:::
## 1. Install the CLI globally
You still need `t4l init` and `t4l start` to configure the project and run the dev server. Install the CLI globally with your package manager:
npmYarnpnpmBun
```bash
npm i -g @tamer4lynx/cli@latest
```
## 2. Install the prebuilt app
Download from the [latest release](https://github.com/tamer4lynx/tamer4lynx/releases/latest):
![](/assets/icon-ios.svg)iOS![](/assets/icon-android.svg)Android**Simulator** — download `TamerDevApp-Simulator-<version>.zip`, unzip, install on a booted simulator:
```bash
unzip TamerDevApp-Simulator-<version>.zip
xcrun simctl install booted ./TamerDevApp.app
xcrun simctl launch booted com.nanofuxion.tamerdevapp
```
**Device** — download `TamerDevApp-Device-Unsigned-<version>.zip`. The `.app` ships unsigned to keep the release portable. Re-sign with your own Apple identity, embed a provisioning profile that matches `com.nanofuxion.tamerdevapp` (or rewrite the bundle id), wrap as IPA, install. Step-by-step in the bundled `SIGN-AND-INSTALL.txt`.
## 3. Initialize your Lynx project
From your Lynx project root, run the interactive setup once:
```bash
t4l init
```
The wizard detects or creates a Lynx app, writes `tamer.config.json`, installs the selected Tamer package stack, injects `pluginTamer()` into `lynx.config.*` when possible, and wires TypeScript ambient component types when a `tsconfig.json` is present. If the Lynx app is nested, it also configures the root as a workspace and installs from the root so dependencies are shared.
## 4. Run the dev server
From your Lynx project root:
```bash
t4l start
```
The dashboard prints a QR. Scan it inside the prebuilt Tamer Dev App — bundles hot-reload over the dev server. Done.
If the configured/default port is already in use, `t4l start` automatically tries the next port and shows that the active port differs from the configured/default port.
## When this flow runs out
Switch to the **Dev Client** flow if you need any of:
- A **custom native module** or **custom element** that isn't shipped in the prebuilt app.
- A different **bundle id**, app icon, or signing identity (e.g. shipping to TestFlight under your own team).
- A **smaller binary** that includes only the Tamer packages your app actually uses.

## CLI and workspace notes

- **Environment files:** `t4l` loads `.env` then `.env.local` beside `tamer.config.json`. Shell / CI exports take precedence. See [Commands → Global](/reference/commands.md) for the full variable list (`ANDROID_KEYSTORE_PASSWORD`, `APP_STORE_CONNECT_API_KEY_PATH`, etc.).
- **Interactive flows:** **`t4l init`** and **`t4l signing`** are Ink wizards. **`t4l start`** is an Ink dashboard: **`r`** rebuild bundle, **`l`** toggle logs, **`q`** quit.
- **TypeScript:** With **`syncTamerComponentTypes`** enabled (default in new configs), **`t4l init`** / **`t4l link`** can adjust the Lynx app `tsconfig.json` and generate **`.tamer/tamer-components.d.ts`**. Details: [Ambient types](/packages/tooling/tamer-ambient-types.md).

Worth exploring next: [tamer-router](/packages/core/tamer-router.md) (back handling), [tamer-webview](/packages/platform/tamer-webview.md), [tamer-local-storage](/packages/platform/tamer-local-storage.md), [tamer-dev-client](/packages/core/tamer-dev-client.md).

## Next steps

- [Native Modules & Elements](/guide/native-modules.md) — author your own Tamer-compatible native package
- [Configuration Reference](/guide/configuration.md) — `tamer.config.json`, `tamer.config.ts`, `lynx.ext.json`
- [Commands](/reference/commands.md) — full CLI reference
- [tamer-dev-client](/packages/core/tamer-dev-client.md) — Dev launcher API and setup

## Lynx ecosystem

- [Lynx](https://lynxjs.org/) — Cross-platform rendering engine
- [Rspeedy](https://lynxjs.org/rspeedy/) — Build tool for Lynx
- [ReactLynx](https://lynxjs.org/react/) — React on Lynx

## Framework agnostic

Tamer4Lynx is conceptually Lynx framework agnostic. Native modules and tooling work with any Lynx binding, including [miso-lynx](https://lynxjs.haskell-miso.org/) and [VueLynx](https://vue.lynxjs.org/) (`@lynx-js/vue`). Some packages like **tamer-router** are specifically designed for `@lynx-js/react`.
