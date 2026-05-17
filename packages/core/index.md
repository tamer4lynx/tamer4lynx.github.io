# Core packages

Routing, dev tooling, host templates, Rsbuild integration, and app shell primitives used across most Tamer apps.

| Package | Role |
|---------|------|
| [tamer-dev-client](/packages/core/tamer-dev-client) | Dev launcher (QR, HMR, recent servers, compatibility checks) |
| [tamer-host](/packages/core/tamer-host) | Lynx host templates — usually auto-installed by `t4l create`; add manually only for existing projects |
| [tamer-navigation](/packages/core/tamer-navigation) | Native stack transport (`TamerNav` push/pop/dispatch) — the foundation `tamer-router` builds on; use directly for custom coordinators |
| [tamer-router](/packages/core/tamer-router) | File-based routing, Stack/Tabs, system back hooks, cross-spoke state bridge |
| [tamer-plugin](/packages/core/tamer-plugin) | Rsbuild plugin merging `tamer.config` |
| [tamer-app-shell](/packages/core/tamer-app-shell) | AppBar, TabBar, Content, navigation chrome |

**`t4l add-core`** installs everything needed for a production app: host templates, plugin, navigation, router, app-shell, screen, insets, system-ui, icons, transports, and env. **`t4l add-dev`** is a superset that also adds the dev launcher packages needed in an app (`tamer-dev-client`, `tamer-linking`).
