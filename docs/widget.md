# Widget contract v1

A widget is a card on the Command Deck's 6-column grid. Users add, move, resize,
configure and remove widget *instances*; one widget *type* can have several
instances when `multi` is true (each with its own settings).

## Manifest (`manifest.json`)

```json
{
  "contract": "helm.widget/1",
  "id": "jira-board",
  "name": "Jira board",
  "version": "0.1.0",
  "description": "Open issues on one Jira board, grouped by status.",
  "category": "Work",
  "icon": "grid",
  "size": { "w": 2, "h": 4, "minW": 2, "minH": 3, "maxW": 6, "maxH": 8 },
  "multi": true,
  "settings": [ ... ],
  "connections": [ ... ],
  "permissions": { "write": false },
  "entry": "index.html",
  "refresh": { "intervalSeconds": 300 }
}
```

| Field | Rules |
| --- | --- |
| `contract` | Exactly `helm.widget/1` |
| `id` | `^[a-z][a-z0-9-]{1,63}$`, unique per origin. Generated widgets are prefixed `user-` |
| `name` | ≤ 60 characters, shown as the panel title (uppercased by the theme) |
| `version` | Semver |
| `description` | ≤ 240, shown in the widget library |
| `category` | `Overview` \| `Work` \| `Knowledge` \| `Agents` \| `Utility` |
| `icon` | A Helm icon name (`ICON_NAMES` in `packages/contracts/src/icons.ts`) |
| `size` | Grid units. `w` 1–6 columns, `h` ≥ 1 rows (row = 76px + 14px gap). Defaults ≤ 6×8 |
| `multi` | Optional, default `false` |
| `settings` | Optional. See [settings.md](settings.md) |
| `connections` | Optional. See [connection.md](connection.md) |
| `permissions.write` | `true` only if the widget calls write operations. Helm asks the user to confirm each write |
| `entry` | HTML file inside the bundle (sandboxed widgets only) |
| `refresh.intervalSeconds` | Optional, ≥ 30. Helm re-sends `refresh` events; widgets may also refresh on demand |

## Anatomy and states

Helm draws the frame: panel, header (marker, title, code, settings, remove, drag
grip) and resize handle. The widget renders **only the body**. Every widget must
handle these states:

| State | Show |
| --- | --- |
| Unconfigured | "Configure this widget" + button (bridge `openSettings()`) |
| Loading | Skeleton rows using `--c-surface-hover`; no spinners longer than 1s without text |
| Empty | One muted sentence explaining what will appear |
| Error | Message + Retry. Use `error.retry` from TDT results to decide wording |
| Ready | Content. Must stay usable from its `minW`×`minH` to its max size |

Use container queries (`@container`) for layout changes; the body is a size container.

## Bridge

Sandboxed widgets talk to Helm with `postMessage`. Helm injects a tiny client
exposing `helm`:

```js
helm.ready()                                  // after first render
helm.on('settings', (settings) => ...)        // initial + on change
helm.on('theme', (tokens) => ...)             // CSS variables already applied to :root
helm.on('refresh', () => ...)
const result = await helm.call('tdt', 'tdt_workspace_status', {})   // connection id, operation, args
helm.openSettings()
helm.openTab({ kind: 'project', id })         // ask Helm to open a tab
helm.startChat('prompt text')                 // hand-off to Claude/Codex
helm.resize({ h: 5 })                         // request, Helm may refuse
```

- Calls are refused unless the connection and operation are declared in the manifest.
- Messages are validated (source window, current instance id, size ≤ 256 KiB).
- The iframe has `sandbox="allow-scripts"` (opaque origin) and a CSP denying
  network, frames, forms, popups and navigation. No `localStorage`; persist via settings.

Built-in widgets are React components that receive the same capabilities through
`WidgetContext` (planned; not yet defined in `packages/contracts/src/`).

## Style

Follow [theme.md](theme.md). In short: tokens only; panel chrome is Helm's; body
text `--fs-md`, labels `.hud-label`; numbers in `--font-display`; signal colours only
for meaning (red critical, amber attention, green nominal).
