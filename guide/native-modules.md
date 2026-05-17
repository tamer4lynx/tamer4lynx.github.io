# Native Modules & Elements

Tamer wraps Lynx's native module / custom element APIs with autolinking (`tamer.config.json` + `t4l link`) so consumers don't manually edit Gradle / Podfile. This page shows how to author a Tamer-compatible native package: a JS-callable native module, a custom UI element, and an optional build-time plugin via `tamer.config.ts`.

For the underlying Lynx APIs, see the upstream guides:

- [Lynx — Use Native Modules](https://lynxjs.org/guide/use-native-modules.html)
- [Lynx — Custom Native Component](https://lynxjs.org/guide/custom-native-component.html)

## Package layout

A typical Tamer native package mirrors `tamer-icons` / `tamer-linking`:

```
my-package/
├── package.json
├── tamer.config.ts          # optional — bundler plugin (see below)
├── lynx.ext.json            # autolink hint
├── src/                     # JS surface (TS compiled to dist/)
├── android/
│   ├── build.gradle.kts
│   └── src/main/kotlin/com/yourorg/mypackage/MyModule.kt
└── ios/
    └── mypackage/
        ├── mypackage.podspec
        └── mypackage/Classes/MyModule.swift
```

After `t4l add my-package` + `t4l link`, the consumer's iOS Podfile picks up the podspec and the consumer's Android Gradle picks up `:my-package`. No manual wiring.

## Native modules (JS-callable functions)

A native module exposes synchronous or callback-style methods to JS via `NativeModules.MyModule`. Tamer doesn't add anything new on top of Lynx — register your `LynxModule` class and the autolinker handles registration.

![](/assets/icon-ios.svg)iOS (Swift)![](/assets/icon-android.svg)Android (Kotlin)`ios/mypackage/mypackage/Classes/MyModule.swift`:
```swift
import Foundation
import Lynx

@objcMembers
public final class MyModule: NSObject, LynxModule {
    @objc public static var name: String { "MyModule" }

    @objc public static var methodLookup: [String: String] {
        [
            "ping": NSStringFromSelector(#selector(ping(_:))),
        ]
    }

    @objc public override init() { super.init() }
    @objc public init(param: Any) { super.init() }

    @objc func ping(_ callback: @escaping (String) -> Void) {
        callback("pong")
    }
}
```

JS surface (`src/index.ts`):

```ts
declare const NativeModules: {
  MyModule?: { ping(cb: (msg: string) => void): void }
}

export function ping(): Promise<string> {
  return new Promise((resolve) => {
    const mod = NativeModules?.MyModule
    if (!mod?.ping) return resolve('')
    mod.ping((msg) => resolve(msg))
  })
}
```

In a consumer Lynx component:

```tsx
import { ping } from '@yourorg/my-package'

useEffect(() => {
  ping().then((msg) => console.log(msg)) // "pong"
}, [])
```

## Custom elements (native UI primitives)

Custom elements register a JSX intrinsic backed by a native view (Android `LynxUI<T>`, iOS `LynxUI` subclass). Useful when CSS/Lynx-built-ins can't render what you need — icon glyphs, native maps, camera previews, charts.

![](/assets/icon-ios.svg)iOS (Obj-C)![](/assets/icon-android.svg)Android (Kotlin)Loosely modeled on `tamer-icons/ios/.../TamerIconElement.m`:
```objc
#import "MyBadgeElement.h"
#import <Lynx/LynxPropsProcessor.h>

@interface MyBadgeHostView : UIView
@property(nonatomic, strong) UILabel *label;
@end

@implementation MyBadgeHostView
- (instancetype)init {
  if ((self = [super initWithFrame:CGRectZero])) {
    _label = [UILabel new];
    _label.textAlignment = NSTextAlignmentCenter;
    [self addSubview:_label];
  }
  return self;
}
- (void)layoutSubviews { [super layoutSubviews]; self.label.frame = self.bounds; }
@end

@implementation MyBadgeElement
- (UIView *)createView { return [MyBadgeHostView new]; }

LYNX_PROP_SETTER("text", setText, NSString *) {
  ((MyBadgeHostView *)self.view).label.text = value ?: @"";
}
@end
```

Augment the JSX namespace so consumers get autocomplete (`src/jsx.ts`):

```ts
/// <reference types="@lynx-js/types" />
import type { ViewProps } from '@lynx-js/types'

export type MyBadgeProps = { text: string } & ViewProps

declare module '@lynx-js/types' {
  interface IntrinsicElements {
    'my-badge': MyBadgeProps
  }
}
```

Consumer:

```tsx
<my-badge text="42" style={{ width: '40px', height: '40px' }} />
```

## Build-time plugin via `tamer.config.ts`

Some packages need to participate in the **consumer's bundler** — fetch fonts, generate routes, inject `define`s, copy assets. Ship an rsbuild plugin from your package and re-export it from `tamer.config.ts`. Tamer's plugin loader picks it up when the consumer imports your package.

Real example — `packages/tamer-icons/src/plugin.ts` downloads Material Icons + Font Awesome on first build, caches them, and copies them into the consumer's `fonts/` directory:

```ts
import fs from 'fs'
import path from 'path'
import type { RsbuildPlugin } from '@rsbuild/core'
import { MATERIAL_ICONS_URL, FONTAWESOME_SOLID_URL } from './fonts'

async function fetchToBuffer(url: string): Promise<Buffer> {
  const res = await fetch(url)
  if (!res.ok) throw new Error(`Failed to fetch ${url}: ${res.status}`)
  return Buffer.from(await res.arrayBuffer())
}

async function ensureFonts(pkgDir: string): Promise<void> {
  const fontsDir = path.join(pkgDir, 'fonts')
  fs.mkdirSync(fontsDir, { recursive: true })
  const cacheDir = path.join(pkgDir, '.cache', 'tamer-icons')
  fs.mkdirSync(cacheDir, { recursive: true })

  const materialPath = path.join(fontsDir, 'MaterialSymbolsOutlined.ttf')
  const materialCache = path.join(cacheDir, 'MaterialSymbolsOutlined.ttf')
  if (!fs.existsSync(materialCache)) {
    fs.writeFileSync(materialCache, await fetchToBuffer(MATERIAL_ICONS_URL))
  }
  fs.copyFileSync(materialCache, materialPath)
  // ...repeat for other fonts
}

export function pluginTamerIcons(): RsbuildPlugin {
  return {
    name: 'tamer-icons',
    async setup(api) {
      const pkgDir = path.resolve(__dirname, '..')
      if (fs.existsSync(path.join(pkgDir, 'package.json'))) {
        await ensureFonts(pkgDir)
      }
    },
  }
}
```

Key design notes:

- **Cache before fetch.** First build downloads, subsequent builds copy from cache. Offline builds keep working.
- **Run side effects in `setup`.** rsbuild calls it once per build; safe place for IO.
- **No mutation of consumer source.** Write into your own package directory or use `api.transform` / `api.modifyRsbuildConfig` instead of editing arbitrary files.

## Wiring through `tamer.config.json`

The consumer adds your package; autolinking handles native registration. JS-side bundler integration goes through `tamer.config.ts`:

```ts
// consumer-app/tamer.config.ts
import { pluginTamerIcons } from '@tamer4lynx/tamer-icons/plugin'

export default {
  plugins: [pluginTamerIcons()],
}
```

The Tamer CLI loads `tamer.config.ts` (if present) and passes the plugin list through to rspeedy/rsbuild.

## See also

- [tamer-plugin](/packages/core/tamer-plugin.md) — the Tamer-side plugin machinery that picks up `tamer.config.ts`
- [Configuration](/guide/configuration.md) — full config schema
- [Lynx — Use Native Modules](https://lynxjs.org/guide/use-native-modules.html)
- [Lynx — Custom Native Component](https://lynxjs.org/guide/custom-native-component.html)
