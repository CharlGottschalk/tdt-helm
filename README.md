# Helm stack

A [ThisDamnThing](https://usetdt.com) stack for making [Helm](https://usetdt.com) Command Deck
plugins from a prompt. It installs two skills:

- `tdt-helm-widget`: a dashboard widget (card) plugin;
- `tdt-helm-tool`: a tool plugin (a tabbed app such as a kanban board).

Both skills scaffold an npm plugin in `work/helm/plugins/<id>/` with
`npx @usetdt/helm-sdk create`, read the contracts bundled with the SDK, fill in
the plugin, and build and validate it with `npm run build`. Installing the plugin
into Helm (`helm plugin link`) happens only when you ask; you review, approve and
enable it in Helm under Settings → Plugins.

## Requirements

- ThisDamnThing with an initialized workspace.
- Node.js 20+ with npm (`npx @usetdt/helm-sdk`).
- Helm, to install and use the plugins.

## Install

Ask your agent to install `tdt-helm` with `/tdt-install-stack`, or from a local
checkout:

```sh
tdt stack validate ./tdt-helm
tdt --workspace ./workspace stack install ./tdt-helm
```

## Contents

```text
stack.json                          manifest and marketplace metadata
skills/tdt-helm-widget/SKILL.md     widget skill
skills/tdt-helm-tool/SKILL.md       tool skill
```

The Helm contracts, templates and CLI live in `@usetdt/helm-sdk`, not here.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
