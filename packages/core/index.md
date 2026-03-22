# Core packages

Routing, dev tooling, host templates, Rsbuild integration, and app shell primitives used across most Tamer apps.

| Package | Role |
|---------|------|
| [tamer-dev-client](/packages/core/tamer-dev-client) | Dev launcher (QR, HMR, recent servers, compatibility checks) |
| [tamer-host](/packages/core/tamer-host) | Production Lynx host templates for existing native apps |
| [tamer-router](/packages/core/tamer-router) | File-based routing, Stack/Tabs, system back hooks |
| [tamer-plugin](/packages/core/tamer-plugin) | Rsbuild plugin merging `tamer.config` |
| [tamer-app-shell](/packages/core/tamer-app-shell) | AppBar, TabBar, Content, navigation chrome |

Use **`t4l add-core`** to install the default UI stack (includes app-shell, router, and related packages). Use **`t4l add-dev`** for the dev-client stack.
