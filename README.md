# Helm stack

> **Beta.** This stack, Helm and the Helm SDK are in beta (0.x). Expect bugs, and changes
> between versions. Report problems in [GitHub issues](https://github.com/CharlGottschalk/tdt-helm/issues).

A [ThisDamnThing](https://usetdt.com) stack that makes [Helm](https://helm.usetdt.com)
Command Deck plugins from a prompt. It installs three skills:

- `tdt-helm-widget` makes a widget plugin: a card on the Helm dashboard.
- `tdt-helm-tool` makes a tool plugin: an application in its own Helm tab, for
  example a kanban board.
- `tdt-helm-theme` makes a theme plugin: colours, fonts and shapes for all of Helm.
  A theme is token values only.

Each skill makes an npm plugin in `work/helm/plugins/<id>/` with
`npx @usetdt/helm-sdk create`. The skill reads the contracts in the SDK, writes the
plugin, and builds and validates it with `npm run build` (a theme: `npm run validate`).
The agent installs the plugin into Helm (`helm plugin link`) only if you ask. You
review and approve the plugin in Helm, under **System settings** → **Plugins**, and
enable a widget or tool for each workspace.

The [usage guide](docs/usage.md) gives the full procedure.

## Requirements

- ThisDamnThing with an initialized workspace.
- Claude Code.
- Node.js 20 or later, with npm (`npx @usetdt/helm-sdk`, version 0.2.0 or later).
- [Helm](https://www.npmjs.com/package/@usetdt/helm), to install and use the plugins:
  `npm install -g @usetdt/helm`.

## Install

From the [stacks marketplace](https://stacks.usetdt.com/tdt-helm), ask your agent:
`/tdt-install-stack tdt-helm`.

Or install it from a local checkout:

```sh
tdt stack validate ./tdt-helm
tdt --workspace ./workspace stack install ./tdt-helm
```

Replace `./workspace` with the path of your workspace.

## Contents

```text
stack.json                          manifest and marketplace metadata
skills/tdt-helm-widget/SKILL.md     widget skill
skills/tdt-helm-tool/SKILL.md       tool skill
skills/tdt-helm-theme/SKILL.md      theme skill
docs/usage.md                       usage guide
```

The Helm contracts, templates and CLI are in `@usetdt/helm-sdk`, not in this stack.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
