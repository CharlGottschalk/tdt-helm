# Settings contract v1

Widgets and tools can declare **per-instance settings**: the values that make two
instances of the same type different, such as which Jira board a widget shows or
which project a kanban tool tracks. Helm renders the settings form; the extension
receives the validated values.

## Declaring settings

`settings` is an array of fields in the manifest. Maximum 24 fields.

```json
"settings": [
  { "id": "board", "type": "select", "label": "Board", "required": true,
    "options": { "from": "connection", "connection": "jira", "call": "boards" } },
  { "id": "columns", "type": "choice", "label": "Columns", "multiple": true,
    "options": ["To do", "In progress", "Review", "Done"], "default": ["To do", "In progress"] },
  { "id": "mine", "type": "checkbox", "label": "Only my issues", "default": true },
  { "id": "limit", "type": "number", "label": "Max issues", "min": 1, "max": 100, "step": 1, "default": 20 },
  { "id": "jql", "type": "text", "label": "Extra JQL filter", "placeholder": "labels = ui", "multiline": false }
]
```

### Common properties

| Property | Type | Notes |
| --- | --- | --- |
| `id` | string | `^[a-zA-Z][a-zA-Z0-9_-]{0,63}$`, unique within the manifest |
| `type` | `text` \| `select` \| `choice` \| `checkbox` \| `number` | |
| `label` | string ≤ 120 | Shown above the control |
| `help` | string ≤ 400 | Optional helper text below the control |
| `required` | boolean | Defaults to `false`. `checkbox` is never required: `false` is a valid value |
| `default` | per type | Validated like a user value. A default is not consent to act |
| `group` | string ≤ 60 | Optional section heading. Consecutive fields with the same group are grouped |

### Field types

| Type | Value | Extra properties | Control |
| --- | --- | --- | --- |
| `text` | string ≤ 2000 | `placeholder`, `multiline` (bool), `pattern` (regex), `maxLength` | Input or textarea |
| `select` | string | `options` (see below) | Dropdown, one value |
| `choice` | string, or string[] when `multiple` | `options`, `multiple` (bool), `min`/`max` selections | Radio group, or checkbox group when `multiple` |
| `checkbox` | boolean | — | Single checkbox/toggle |
| `number` | finite number | `min`, `max`, `step` (> 0), `unit` (string ≤ 16) | Number input |

### Options

`options` for `select` and `choice` is either:

- a static array of strings (1–100, unique, each ≤ 200), or of
  `{ "value": string, "label": string }` objects; or
- a **dynamic source** `{ "from": "connection", "connection": "<id>", "call": "<call-id>" }`
  that Helm resolves through a connection declared by the same manifest (see
  [connection.md](connection.md)). The call must return `[{ value, label }]`.
  Helm shows a loading state, an error with retry, and keeps a stale value marked
  "unavailable" rather than silently dropping it.

## Rules

- **Never store secrets in settings.** API tokens and passwords belong to the
  connection, which Helm stores outside the workspace (see connection.md). A field
  whose id or label suggests a secret (`token`, `password`, `secret`, `key`) is
  refused by validation.
- Settings are saved per instance with the dashboard layout or tool tab, are
  user-editable, and changes apply immediately (no restart).
- An extension must render something sensible while required settings are missing:
  a short "Configure this widget" prompt with a button that opens settings.
- Validation happens in Helm before values reach the extension. Extensions receive
  only declared ids with values of the declared types.

## Runtime

Helm passes settings in the render context (`ctx.settings`) and pushes updates with
a `settings` bridge event. Built-in React extensions read `useSettings()`; see
`packages/sdk/src/settings.ts`. The form renderer is `apps/helm/src/components/settings/SettingsForm.tsx`.
