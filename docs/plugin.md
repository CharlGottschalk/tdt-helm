# Plugin package contract v1

A Helm plugin is an npm package that carries one or more sandboxed widgets and tools.
Its `package.json` declares them in a `"helm"` field:

```json
{
  "name": "helm-plugin-example",
  "version": "1.0.0",
  "keywords": ["helm-plugin"],
  "helm": {
    "contract": 1,
    "widgets": ["widgets/workspace-status"],
    "tools": []
  },
  "devDependencies": { "@usetdt/helm-sdk": "^0.1.0" }
}
```

| Field | Rule |
| --- | --- |
| `helm.contract` | `1` (`PLUGIN_CONTRACT`). Helm refuses versions it doesn't support with a "needs a newer Helm" message. |
| `helm.widgets`, `helm.tools` | Directories relative to the package root, no `..`, each listed once; at least one in total. |

Each listed directory is a sandboxed bundle as described in [README.md](README.md):
`manifest.json` ([widget.md](widget.md) or [tool.md](tool.md)) plus its self-contained
`entry` HTML, with the folder name equal to the manifest `id`. Use the
`helm-plugin` npm keyword so plugins can be found.

Machine check: `validatePluginPackage(pkg)` in `@usetdt/helm-sdk`, schema in
[`schemas/plugin.schema.json`](schemas/plugin.schema.json).

## Writing the bundle

- Style only with theme tokens and the [UI kit](kit.md); both are injected before
  your `<style>`.
- Talk to Helm only through the bridge ([widget.md#bridge](widget.md#bridge)). For
  TypeScript, `import { getHelm, el } from '@usetdt/helm-sdk/frame'` gives the typed
  bridge and an element helper that inserts text safely.
- Render connection data as text (`textContent` or `el`), never as HTML.
- No install scripts: Helm installs plugins with `--ignore-scripts`, so a bundle must
  ship built.

Installing, reviewing and enabling plugins per workspace comes with the plugin host
(task 016); a scaffolding and validation CLI comes with task 015.
