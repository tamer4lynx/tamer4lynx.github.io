# CLI Commands Reference

Commands use the form **`t4l <command> [target] [flags]`**. Target is a platform (`ios`, `android`) or, for create, an extension type (`module`, `element`, `service`, `combo`). All flags support both short and long form (e.g. `-d` or `--debug`).

---

## Global

Environment: On startup, `t4l` loads **`.env`** then **`.env.local`** from the directory containing `tamer.config.json` (walking up from the current working directory). Values are applied only when the variable is **not** already set in the process environment (so CI or shell exports win). Use this for Android signing variables such as `ANDROID_KEYSTORE_PASSWORD` / `ANDROID_KEY_PASSWORD` referenced in `tamer.config.json`, and for **App Store Connect API** variables used with `t4l build ios -p --ipa`: **`APP_STORE_CONNECT_API_KEY_PATH`** (path to the `.p8` key file), **`APP_STORE_CONNECT_API_KEY_ID`**, and **`APP_STORE_CONNECT_ISSUER_ID`**. Optional overrides for those env var *names* live under `ios.appStoreConnect` in `tamer.config.json`. If all three are set, the CLI runs `xcodebuild archive` / `-exportArchive` with **app-store** export (distribution signing + API key auth) instead of zipping a locally signed `.app`.

| Command | Description |
|---------|-------------|
| `t4l` or `t4l init` | Interactive setup: creates `tamer.config.json` in the current directory |
| `t4l add …` | Add `@tamer4lynx/*` to the Lynx project — details below |
| `t4l add-core` | Install the core stack in one command — details below |
| `t4l add-dev` | Install dev-app, dev-client, and their dependencies — details below |
| `t4l update` | Bump every `@tamer4lynx/*` listed in **package.json** to the **highest published semver** — details below |
| `t4l signing [platform]` | Configure Android/iOS signing (interactive; Android can generate a keystore with `keytool`) |
| `t4l --help` | Show help |
| `t4l --version` | Show version |

### Developing the CLI (this repository)

Do not run `node index.ts` — Node ESM does not resolve extensionless `./src/...` imports. From the repo root: `npm run cli -- <args>` (runs `tsx index.ts`), or `npm run build && node dist/index.js <args>`.

---

## `t4l init`

Initialize `tamer.config.json` with an **Ink** interactive wizard: step-by-step prompts with validation (Android package / iOS bundle ID), optional `$ENV` resolution for the Android SDK path, and a confirmation when reusing Android values for iOS. Writes `tamer.config.json` (including **`syncTamerComponentTypes`: true** by default). For **TypeScript**, when a `tsconfig.json` exists (project root or under `lynxProject`), it may **flatten project references** (TS6310), then generate **`.tamer/tamer-components.d.ts`** (triple-slash references to installed `@tamer4lynx/*` ambient type exports) and **ensure `include`** for that file. Broad globs such as `node_modules/@tamer4lynx/tamer-*/src/**/*.d.ts` are removed when present.

No flags.

---

## `t4l signing [platform]`

Configure Android and iOS release signing in `tamer.config.json` (Ink interactive wizard).

| Argument | Description |
|----------|-------------|
| *(none)* | Choose Android, iOS, or both |
| `android` | Android only |
| `ios` | iOS only |

**Android:** Choose **Generate a new release keystore** (requires a JDK: `keytool` on `PATH` or `JAVA_HOME`) or **Use an existing keystore**. Generation runs `keytool -genkeypair` (RSA 2048, 10000-day validity) and writes the keystore under a path you choose (default `android/release.keystore`). Passwords are referenced by env var names in `tamer.config.json`; on **generate**, the wizard **appends** `ANDROID_KEYSTORE_PASSWORD` / `ANDROID_KEY_PASSWORD` (or names you choose) to **`.env.local`** if it exists, else **`.env`** if that file exists, else it creates **`.env.local`**—each new line only; existing keys are not overwritten. The CLI loads **`.env`** then **`.env.local`** at startup (see Global). Gradle can still use an optional `android/signing.properties` if you maintain it yourself; the wizard does not write it.

**iOS:** Prompts for Development Team ID and optional code-sign identity / provisioning profile specifier.

After setup, use `t4l build android -p` / `t4l build ios -p` for production signed builds.

---

## `t4l create <target>`

Create a project or extension. **Target:** `ios` | `android` | `module` | `element` | `service` | `combo`.

| Target | Description |
|--------|-------------|
| `ios` | Create iOS project |
| `android` | Create Android project (host by default; use `-r` for dev-app) |
| `module` | Create Lynx extension with native module only |
| `element` | Create Lynx extension with custom element (JSX preserved) |
| `service` | Create Lynx extension with service only |
| `combo` | Create Lynx extension with module + element + service |

| Flag | Short | Description |
|------|-------|-------------|
| `--debug` | `-d` | For android: create host project (default) |
| `--release` | `-r` | For android: create dev-app project |

Cannot use `--debug` and `--release` together.

**Examples:**

```bash
t4l create ios
t4l create android              # host project
t4l create android -r           # dev-app project
t4l create android --release
t4l create module
t4l create element
t4l create combo
```

---

## `t4l build <platform>`

Build your app. **Platform is required:** `ios` | `android` (one platform per command). With **debug** (`-d`), the dev client (QR scan, HMR) is embedded when `@tamer4lynx/tamer-dev-client` is installed; with **release** (`-r`), the build has no dev client (unsigned); with **production** (`-p`), the build is signed for app store.

**Production (`-p`):** Configure signing first (`t4l signing`, or `t4l signing android` / `t4l signing ios`). If signing is not set up for that platform, the CLI exits with instructions. To build both platforms, run `t4l build android -p` and `t4l build ios -p` separately.

**iOS `-r` / `-p` vs dev-client:** Release/production builds **do not** build or copy `dev-client.lynx.bundle` and use the plain host `ViewController` (main bundle only). You may still see `@tamer4lynx/tamer-dev-client` in the autolinker’s package list because it provides native iOS pods; that is not the same as embedding the dev-client Lynx shell.

**Lynx DevTool:** Debug (`-d`) hosts with `tamer-dev-client` register native DevTool hooks so you can attach the [desktop Lynx DevTool](https://lynxjs.org/guide/devtool.md) over USB. **`-r` / `-p` builds do not enable DevTool** (iOS: `#if DEBUG` only; Android: debug classpath / no-op in release).

**iOS `-p` and `-i`:** Production (`-p`) always builds for **real devices** (`iphoneos`) with code signing enabled (configure via `t4l signing ios`). Signing settings from `tamer.config.json` are passed to `xcodebuild`. With `-i`, the CLI installs using `xcrun devicectl device install app --device <UDID>` (Xcode 15+). If **multiple** physical devices are connected, an interactive **device picker** runs; with **one** device, install proceeds without a prompt. Debug (`-d`) with `-i` still targets the **simulator** (`simctl install`).

**Android `-i`:** Install uses Gradle `install*` with `ANDROID_SERIAL` set when you pick a device, or a single connected device. If **multiple** `adb` devices are connected, an interactive **device picker** runs; with **one** device, install proceeds without a prompt. Launch uses `adb -s <serial> shell am start …`.

**iOS `--ipa`:** With `t4l build ios -p` (and optionally `-i`), **`--ipa`** produces an IPA under `ios/build/ipa-export/`. **Without** App Store Connect env vars (see Global): unsigned build, manual `codesign`, then zip into `Payload` (manual path). **With** `APP_STORE_CONNECT_API_KEY_PATH`, `APP_STORE_CONNECT_API_KEY_ID`, and `APP_STORE_CONNECT_ISSUER_ID` set (and `ios.signing.developmentTeam`), the CLI uses **`xcodebuild archive`** + **`-exportArchive`** with method **app-store** and your `.p8` API key. Use `-p`; `--ipa` alone is invalid.

**Non-interactive CI:** If multiple devices are connected and stdin is not a TTY, install fails with a message to connect a single device.

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--embeddable` | `-e` | — | Output to `embeddable/` for adding LynxView to an existing app. **Requires** `t4l build android --embeddable` (Android only). Runs a **release-style** embeddable build internally; you do **not** need to pass `--release`. When `-e` is present, the CLI only performs the embeddable step. |
| `--debug` | `-d` | default | Debug build with dev client embedded (if tamer-dev-client is installed). |
| `--release` | `-r` | — | Release build without dev client (unsigned). |
| `--production` | `-p` | — | Production build for app store (signed). |
| `--install` | `-i` | — | **Android:** install APK to device. **iOS:** with **debug** (`-d`), install to the **booted simulator**; with **production** (`-p`), install to a **physical device only** via `devicectl` (simulator is never used with `-p`). |
| `--ipa` | — | — | **iOS only** (`t4l build ios -p`): archive and export a signed IPA after the production build. |
| `--clean` | `-C` | — | **Android only:** run Gradle `clean` before building (helps with stubborn caches). No effect on iOS-only builds. |

**Examples:**

```bash
t4l build android
t4l build android -i
t4l build ios -r
t4l build android --release --install
t4l build android -C
t4l build ios -p -i
t4l build ios -p --ipa
t4l build android --embeddable
```

---

## `t4l link [platform]`

Link native modules to the project and sync native dependencies. **Platform:** `ios` | `android` | `both` (optional; default `both`).

- **iOS:** Updates Podfile and `LynxInitProcessor.swift` (imports, module registrations, `DevClientModule.attachSupportedModuleClassNames`), then runs **`pod install`** in the `ios` directory. If `@tamer4lynx/tamer-dev-client` is installed, also writes **`tamer-host-native-modules.json`** into `ios/<AppName>/` (lists JVM-style `moduleClassName` values matching dev server `meta.json`) and registers it in the Xcode project’s **Copy Bundle Resources**.
- **Android:** Updates settings.gradle.kts, app/build.gradle.kts, and generated code, then runs **Gradle sync** (`./gradlew projects`) in the `android` directory.
- **TypeScript / IDE:** When **`syncTamerComponentTypes`** is not `false` in `tamer.config.json`, each platform’s autolink step also regenerates **`.tamer/tamer-components.d.ts`** and ensures **`tsconfig.json`** includes it (see [Ambient types](/packages/tooling/tamer-ambient-types)).

| Flag | Short | Description |
|------|-------|-------------|
| `--silent` | `-s` | Run without output. Useful for CI or postinstall scripts. |

**Examples:**

```bash
t4l link
t4l link android
t4l link --silent
```

**Host apps (`tamer-dev-app`, projects created with `t4l ios create`):** `LynxInitProcessor.swift` from **tamer-dev-client** templates only contains `GENERATED IMPORTS` / `GENERATED AUTOLINK` placeholders. Run `t4l link` from the app root (where `tamer.config.json` lives) so those sections are filled from your `node_modules` `@tamer4lynx/*` packages. In the monorepo, `packages/tamer-dev-app` provides `npm run link:native` after building the CLI (`npm run build` at repo root).

**User-provided native modules:** To ensure your custom native extensions are discovered and linked:
1. Install the package in your workspace root `package.json` (or ensure it's hoisted to the root `node_modules` in monorepos).
2. The package must include `lynx.ext.json` or `tamer.json` with `ios` and/or `android` configuration.
3. Run `t4l link` after adding dependencies to update Podfile/Gradle and native registration code.
4. For iOS, run `pod install` in the `ios/` directory after podspec changes.

**iOS `No podspec found` for a Tamer package:** Often the app depended on **`npm install …@latest`**, where **`latest`** points at an older tarball without `ios/`. Prefer **`t4l add tamer-insets`** (or the relevant package): it resolves the **highest published semver** from the registry, unlike `latest` when that tag lags. Then run **`t4l link ios`** again.

**Xcode reports unknown UUID in `project.pbxproj`:** This can occur after merge conflicts or if the project file was manually edited. Fix by reverting `ios/<AppName>.xcodeproj/project.pbxproj` from git and re-running `t4l bundle ios` or `t4l link ios` to regenerate resource references.

---

## `t4l bundle [platform]`

Build the Lynx bundle and copy it to the native project. **Platform:** `ios` | `android` (optional; omit for both). Runs **autolink** before bundling (iOS: includes `tamer-host-native-modules.json` when dev-client is present — see `t4l link`).

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--debug` | `-d` | default | Debug bundle with dev client embedded (if present) |
| `--release` | `-r` | — | Release bundle without dev client (unsigned) |
| `--production` | `-p` | — | Production bundle for app store (signed) |

---

## `t4l inject <platform>`

Inject tamer-host templates into an existing project. **Platform:** `ios` | `android` (required).

| Flag | Short | Description |
|------|-------|-------------|
| `--force` | `-f` | Overwrite existing files. Without this, existing files are skipped. |

**Examples:**

```bash
t4l inject android
t4l inject ios -f
```

---

## `t4l sync [platform]`

Sync dev client files (TemplateProvider, MainActivity, DevClientManager) from `tamer.config.json`. **Platform:** `android` only (default).

**Note:** This runs automatically during `t4l bundle` and `t4l build`. Use it manually when you've changed `tamer.config.json` (e.g., `devServer.host` or `devServer.port`) and want to update the generated Android files without building or bundling.

**Examples:**

```bash
t4l sync
t4l sync android
```

---

## `t4l start`

Start the dev server with HMR and WebSocket support (Expo-like). Prints bundle URLs, `meta.json`, QR code, and WebSocket endpoint.

**Multiple bundles:** set `paths.lynxAdditionalBundles` in `tamer.config.json` to an array of extra `.lynx.bundle` filenames under the same `paths.lynxBundleRoot` as `paths.lynxBundleFile`. Each is listed under `bundles` in `meta.json` and served from the dev server alongside the primary bundle.

**Keyboard shortcuts** (stdin, TTY): **`r`** rebuild, **`c`** / **Ctrl+L** clear screen, **Ctrl+C** exit.

| Flag | Short | Description |
|------|-------|-------------|
| `--verbose` | `-v` | Show all logs (native + JS). Default shows JS only. |

---

## `t4l add [packages...]`

Add `@tamer4lynx` packages to the Lynx project.


| Argument | Description |
|----------|-------------|
| `packages...` | Package names (e.g. `tamer-auth`, `@tamer4lynx/tamer-auth`). Bare names get `@tamer4lynx/` prefix. |


**Examples:**

```bash
t4l add tamer-auth
t4l add @tamer4lynx/tamer-auth @tamer4lynx/tamer-secure-store
```

---

## `t4l add-core`

Adds the core stack: **tamer-app-shell**, **tamer-screen**, **tamer-router**, **tamer-insets**, **tamer-transports**, **tamer-system-ui**, **tamer-icons**, **tamer-env** (Rspeedy `.env` plugin). Each package is resolved to the **highest published semver** on npm (same as `t4l add` / `t4l add-dev`). No flags.

---

## `t4l add-dev`

Adds the **dev stack**: **tamer-dev-app**, **tamer-dev-client**, and the `@tamer4lynx/*` packages they need (**app-shell**, **icons**, **insets**, **plugin**, **router**, **screen**, **system-ui**). Each is resolved to the **highest published semver** on npm (same idea as `add-core` / `t4l add`). No flags.

---

## `t4l update`

Scans **`dependencies`**, **`devDependencies`**, **`peerDependencies`**, and **`optionalDependencies`** in the Lynx project’s **package.json** for **`@tamer4lynx/*`** names and re-installs each at the **highest published semver** on npm (same resolution as `t4l add`). Skips **`file:`**, **`link:`**, **`portal:`**, and **`workspace:`** specs so local / monorepo links stay unchanged. If nothing matches, add packages first with **`t4l add`** / **`add-core`** / **`add-dev`**. No flags.

If npm reports **`ETARGET`** for **`@tamer4lynx/...@^0.0.n`**, remember that **`^` on 0.0.x** only allows the same patch line (e.g. **`^0.0.3`** is **`<0.0.4`**). A transitive dependency may still pin an older range; published **`@tamer4lynx/*`** packages use **`>=0.0.1`**-style ranges for internal deps so newer **0.0.x** releases stay compatible.

---

**After `t4l add`, `t4l add-core`, `t4l add-dev`, or `t4l update`:** run **`t4l link`** so native modules are wired into iOS/Android.

---

## `t4l codegen`

Generate code from `@lynxmodule` declarations in `.d.ts` files. Run from an extension package root.

No flags.

---

## `t4l autolink-toggle` / `t4l autolink`

Toggle `autolink` on/off in `tamer.config.json`. When enabled, `t4l link` runs after **installing dependencies** (e.g. `postinstall` after `npm install` / `pnpm install` / `bun install`). Does not affect build or bundle commands, which always run link.

No flags.

---

## Platform-first commands

The following form works: **`t4l <platform> <subcommand> [flags]`**.

| Command | Description |
|---------|-------------|
| `t4l android create` | Same as `t4l create android`. Use `-r` or `--release` for dev-app. |
| `t4l android link` | Same as `t4l link android` |
| `t4l android bundle` | Same as `t4l bundle android`. Flags: `-d`, `-r`, `-p`. |
| `t4l android build` | Same as `t4l build android`. Flags: `-d`, `-r`, `-p`, `-i`, `-e`, `-C`. |
| `t4l android sync` | Same as `t4l sync android` |
| `t4l android inject` | Same as `t4l inject android`. Flag: `-f`. |
| `t4l ios create` | Same as `t4l create ios` |
| `t4l ios link` | Same as `t4l link ios` |
| `t4l ios bundle` | Same as `t4l bundle ios`. Flags: `-d`, `-r`, `-p`. |
| `t4l ios build` | Same as `t4l build ios`. Flags: `-d`, `-r`, `-p`, `-i`, `--ipa`, `-e`. |
| `t4l ios inject` | Same as `t4l inject ios`. Flag: `-f`. |

**Note:** Use `t4l link ios` or `t4l link android` (not `--ios` / `--android`).
