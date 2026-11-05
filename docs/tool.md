# Tool contract v1

A **tool** is a full-size, tabbed workspace application that opens beside the Command
Deck, the brain and project tabs: a kanban board, a decision log, a release checklist.
Where a widget is a glanceable card, a tool is where you work.

## Manifest

```json
{
  "contract": "helm.tool/1",
  "id": "kanban",
  "name": "Kanban",
  "version": "0.1.0",
  "description": "Software-production tasks by status; start builds from ready cards.",
  "icon": "grid",
  "multi": true,
  "settings": [
    { "id": "project", "type": "select", "label": "Project", "required": true,
      "options": { "from": "connection", "connection": "tdt", "call": "projects" } },
    { "id": "showCancelled", "type": "checkbox", "label": "Show cancelled", "default": false }
  ],
  "connections": [ { "id": "tdt", "kind": "tdt-mcp", "operations": ["tdt_project_list"] } ],
  "capabilities": ["orchestrate"],
  "permissions": { "write": true },
  "entry": "index.html"
}
```

Same field rules as [widget.md](widget.md), except:

| Field | Rules |
| --- | --- |
| `contract` | Exactly `helm.tool/1` |
| `multi` | `true` lets users open several instances (each tab has its own settings) |
| `capabilities` | Optional host features: `orchestrate` (start build sessions), `terminal` (open a workspace terminal tab), `chat` (hand-off). Undeclared capabilities are refused |
| no `size` | Tools fill the tab viewport |

## Instances and tabs

- Opening a tool creates an instance `{ id, type, settings }` saved with the
  workspace's tab set. Tab title: tool name, plus the value of the first required
  setting when present (e.g. `Kanban · Atlas`).
- Closing a tab keeps the instance in "Recent tools" until the user removes it.
- Tools appear in the left rail's **Tools** group and in the tool library.

## Layout

Helm renders the tab; the tool renders the whole viewport below the tab bar.
Use the standard page header (title, `.hud-label` eyebrow, actions on the right) and
`Panel`s or columns inside. Leave 140px bottom padding: the floating chat dock overlays tools.

## Bridge

Everything in the widget bridge, plus:

```js
helm.orchestrate.build({ projectId, taskId, host })   // requires capability "orchestrate"
helm.sessions.on('change', (sessions) => ...)         // session status for cards
helm.openTerminal({ cwd })                            // requires "terminal"; cwd must be inside the workspace
```

Orchestration calls start host sessions that appear in the right panel; the tool
should reflect their status (for example a card shows "building" while its session runs).
