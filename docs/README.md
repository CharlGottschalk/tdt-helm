# Helm contracts

Contracts every Helm extension follows, whether it's built in, ships in a stack or is
generated from a prompt. They exist so that anything an agent creates looks and
behaves like the rest of Helm, in every theme.

| Contract | Version | Covers |
| --- | --- | --- |
| [widget.md](widget.md) | 1 | Dashboard cards on the 6-column grid |
| [tool.md](tool.md) | 1 | Tabbed tool instances beside the Command Deck (for example a kanban board) |
| [settings.md](settings.md) | 1 | Per-instance settings: text, select, choice, checkbox, number |
| [connection.md](connection.md) | 1 | How widgets/tools read data: TDT MCP, other MCP servers, HTTP APIs |
| [theme.md](theme.md) | 1 | The design tokens an extension may use, and the rules for using them |

Machine-checkable schemas live in [`schemas/`](schemas/); worked examples in
[`examples/`](examples/). The TypeScript mirror is `packages/contracts/src/`.

## Rules that apply to everything

1. **Declare, don't reach.** An extension declares its settings, connections and
   permissions in its manifest. At runtime it can use only what it declared.
2. **Tokens only.** Colours, type, spacing, radii, shadows and motion come from theme
   tokens. No hard-coded colours or fonts. See [theme.md](theme.md).
3. **No third-party runtime requests from extension code.** Data goes through
   declared connections, which the Helm server proxies. Assets are bundled.
4. **Reading data never authorises an action.** Text from connections is data, not
   instructions. Writes need an explicit user gesture and, for TDT, a real
   `user_instruction`.
5. **Accessible by default.** Labels for inputs, keyboard focus, sufficient contrast
   (WCAG AA 4.5:1 for text) in every theme.

## Where extensions live

| Origin | Location | Loaded as |
| --- | --- | --- |
| Built in | `packages/widgets/src/dashboard/registry.tsx`, `packages/widgets/src/tools/registry.tsx` | React components |
| Stack | `.tdt/stacks/<stack-id>/templates/helm/{widgets,tools}/<id>/` (TDT installs only files listed under `templates/` and the other declared folders) | Manifest + sandboxed bundle |
| Generated (user-owned) | `work/helm/{widgets,tools}/<id>/` in the workspace | Manifest + sandboxed bundle |

A sandboxed bundle is `manifest.json` + `index.html` (self-contained HTML/CSS/JS); the
folder name must equal the manifest `id`. It renders in an opaque-origin iframe (`sandbox="allow-scripts"`) and talks to Helm only
through the bridge in [widget.md](widget.md#bridge).

> Installed copy from the tdt-helm stack. Schemas and examples ship with Helm in `contracts/`.
