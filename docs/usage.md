# Using the Helm stack

The Helm stack adds three skills to your [ThisDamnThing](https://usetdt.com) (TDT)
workspace. With these skills, the agent makes plugins for the Helm Command Deck from
your description:

| Skill | Makes |
| --- | --- |
| `tdt-helm-widget` | A **widget**: a card on the Helm dashboard |
| `tdt-helm-tool` | A **tool**: an application in its own Helm tab, for example a kanban board |
| `tdt-helm-theme` | A **theme**: a set of colours, fonts and shapes for all of Helm |

The agent makes each plugin as an npm package in your workspace, in
`work/helm/plugins/<id>/`. It builds and validates the plugin. You then install the
plugin into Helm, and you review and approve it there. You enable a widget or a tool
for each workspace. A theme needs no enable step.

## Requirements

- TDT with an initialized workspace.
- Claude Code. The stack supports Claude Code only.
- Node.js 20 or later, with npm.
- Helm, to install and use the plugins: `npm install -g @usetdt/helm`. See the
  [Helm user guides](https://helm.usetdt.com/docs) for Helm's own requirements.
- The skills use the Helm SDK, [`@usetdt/helm-sdk`](https://www.npmjs.com/package/@usetdt/helm-sdk)
  0.2.0 or later, through `npx`. You do not install it yourself.

## Install the stack

From the [stacks marketplace](https://stacks.usetdt.com/tdt-helm), ask your agent in
your workspace: `/tdt-install-stack tdt-helm`.

To install from a local checkout instead, run these commands from the folder that
contains the `tdt-helm` checkout and your workspace. Replace `./workspace` with the
path of your workspace.

```sh
tdt stack validate ./tdt-helm
tdt --workspace ./workspace stack install ./tdt-helm
tdt --workspace ./workspace stack list
```

1. `stack validate` checks the stack. If it reports a problem, do not install the stack.
2. `stack install` installs the three skills into the workspace.
3. `stack list` shows `tdt-helm`, if the installation is complete.

After the installation, start Claude Code again in your workspace. The skills are then
available as `/tdt-helm-widget`, `/tdt-helm-tool` and `/tdt-helm-theme`.

The stack installs only these three skills. It adds no hooks and no brain knowledge.

## Make a widget

1. In Claude Code, in your workspace, type `/tdt-helm-widget` and describe the widget.
2. Answer the questions of the agent. The agent asks about these items:
   - What the widget shows.
   - Where the data comes from: TDT, another MCP server or an HTTP service.
   - Which settings each widget instance has, for example a project or a limit.
   - The default size.
   - Whether the widget changes data. Read-only widgets are better.
3. The agent makes the plugin in `work/helm/plugins/<id>/`, builds it and validates it.
   The id contains lowercase letters, digits and dashes, for example `user-my-widget`.
4. The agent reports the result, and any contract rule that it could not satisfy.

To answer the questions in a TDT form instead of in chat, add "use ui" to your request.

Never give the agent passwords or API tokens. A widget that uses an HTTP service gets
its credentials from a service profile in Helm. You add that profile yourself in Helm,
under **System settings** → **Connections**.

## Make a tool

1. In Claude Code, in your workspace, type `/tdt-helm-tool` and describe the tool.
2. Answer the questions of the agent. The agent asks about these items:
   - The job of the tool.
   - Its data sources and its settings.
   - Whether you can open the tool several times, each with its own settings.
   - Which Helm features it needs: `orchestrate` to start builds, `terminal` or `chat`.
3. The agent makes, builds and validates the plugin, as for a widget.

A tool with `orchestrate` starts builds only through the TDT software-production build
skill. The tool never marks tasks as done. The build skill and you control the task
status.

## Make a theme

1. In Claude Code, in your workspace, type `/tdt-helm-theme` and describe the look.
2. Answer the questions of the agent. The agent asks about these items:
   - The look, in a few words.
   - The accent colours and the background and panel colours.
   - Light or dark, the corners and the fonts.
   - Whether the rails (the sidebars, the tab bar and the chat dock) look different
     from the content, as in Monolith.
3. The agent makes the plugin in `work/helm/plugins/<id>/`. A theme has no code: the
   agent writes only token values, in `themes/<id>/`. It validates the theme with
   `npm run validate`.
4. The agent reports each contrast warning, with the two colours and their ratio, and
   offers a fix. Text below WCAG AA (4.5:1) is allowed, but Helm shows
   **Below AA contrast** on the theme. You decide whether to keep it.

A theme can change only tokens. It cannot add background images, animations or font
files. See `theme.md` in the SDK.

## Change a plugin

Give the skill the id of the existing plugin, and describe the change. The agent edits
the plugin in its folder, and builds and validates it again.

You can also edit the plugin yourself. Edit only the files in `src/<id>/`, or in
`themes/<id>/` for a theme. The build makes `dist/` again. A theme has no build. In the plugin folder, run:

```sh
npm run build      # build and validate the plugin
npm run validate   # validate without a build
npm run dev        # build again after each change in src/
```

## Install a plugin into Helm

The agent installs the plugin into Helm only if you ask. Otherwise it gives you the
command.

1. Link the plugin into Helm. Replace the path with the absolute path of the plugin
   folder.

   ```sh
   helm plugin link /abs/path/to/workspace/work/helm/plugins/<id>
   ```

2. In Helm, open **System settings** → **Plugins**.
3. Find the plugin under **Installed**. Read its declarations: the connections, the
   calls, the writes and the capabilities.
4. If you agree, click **Approve these declarations**.
5. For a widget or a tool, turn on **Enabled in {workspace}**. A theme does not have
   this step.

The result:

- A widget appears in the **Widget library**, in its category.
- A tool appears in the **Tools** group of the left rail, and in the tool library.
- A theme appears in **System settings** → **Appearance** → **Theme**, in every
  workspace. Click it to use it.

Only you can approve and enable a plugin. The agent does not do these steps.

### After a change

Helm reads a linked plugin directly from its folder. After a build, reload the widget
or tool in Helm. If the manifest or a theme token file changed, Helm shows
**Needs review** again. Approve
the plugin again before you use it.

## What the skills do not do

- They do not change Helm, other plugins, or files outside the plugin folder.
- They do not ask for credentials, and they do not keep credentials.
- They do not approve or enable plugins in Helm, and they do not select a theme.
- They do not save the plugin as brain knowledge. The plugin is your work in `work/`.

## Troubleshooting

### The SDK is not found

If `npx @usetdt/helm-sdk create` reports `E404` or a network error, check your
connection to the npm registry and any npm proxy or registry settings
(`npm config get registry`).

### The build reports problems

`npm run build` validates the plugin against the Helm contracts. It reports each
problem, for example a hard-coded colour or a call that the manifest does not declare.
Ask the agent to correct the problems, or correct them in `src/<id>/`. Then build
again.

### Helm shows "Needs a newer Helm"

The plugin uses a contract version that your Helm does not support. Update Helm.

### `helm` is not found

Install Helm with `npm install -g @usetdt/helm`. If you run Helm from its source
code, run the command from the root of the Helm repository as `npx helm …`.

## Reference

The Helm contracts, templates and CLI are in the `@usetdt/helm-sdk` package, not in
this stack. The skills read the contracts from the SDK that the plugin installs, in
`node_modules/@usetdt/helm-sdk/docs/`.
