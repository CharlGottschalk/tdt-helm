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

## Installing

```bash
helm plugin add helm-plugin-example        # npm name or range; also a git URL
helm plugin link ./my-plugin               # a local folder, linked for development
helm plugin list
helm plugin remove helm-plugin-example
```

Or use Settings → Plugins. Plugins install once per Helm into `~/.helm/plugins/` (its
own `package.json` and lockfile), always with `--ignore-scripts`. Installing never
enables anything:

1. **Review.** Helm shows every widget and tool the plugin declares, with its
   connections, operations, writes and capabilities.
2. **Approve.** The approval covers exactly those declarations (a digest of the
   version, the `"helm"` field and every manifest). An update or a changed manifest
   needs a new approval; until then its widgets show an error instead of loading.
3. **Enable** it per workspace. Only enabled, approved plugins reach a workspace's
   library.

A plugin whose `helm.contract` this Helm doesn't support shows "needs a newer Helm".

## The `helm-sdk` CLI

```bash
npx @usetdt/helm-sdk create widget my-widget   # or: create tool my-tool
cd my-widget
npx helm-sdk build                             # src/<id>/ → dist/<id>/, then validate
npx helm-sdk validate                          # check without building
npx helm-sdk dev                               # build, then rebuild on every change in src/
helm plugin link .                             # once; review and enable in Settings → Plugins
```

`create` scaffolds a package whose folder name is the plugin id: `package.json` (the
`"helm"` field lists `dist/<id>`), and `src/<id>/` with `manifest.json`, `index.html`
and `main.js`. Edit `src/`; `dist/` is generated, and ships with the package because
Helm never runs install scripts.

`build` copies each manifest and inlines the entry's local `<script src>` (bundled, so
`import … from '@usetdt/helm-sdk/frame'` works) and `<link rel="stylesheet">` into one
self-contained HTML file. A directory listed outside `dist/` is shipped as is.

`validate` checks the `"helm"` field and every manifest against the contracts, plus
what the frame would refuse or the contracts forbid in the entry: script or
stylesheet files, non-`data:` images and CSS `url()`s, hard-coded colours or fonts in
`<style>` and `style=""`, and `helm.call()`s (with literal arguments) to connections or
operations the manifest doesn't declare. It exits non-zero on any problem.

`dev` builds, validates and rebuilds on every change, and tells you whether Helm is
running and the plugin is linked. Helm reads a linked plugin's files directly, so
reload the widget or tool to see a rebuild; a changed manifest needs a new approval.
