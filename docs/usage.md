# Using the Helm stack

The Helm stack adds two skills to your [ThisDamnThing](https://usetdt.com) (TDT)
workspace. With these skills, the agent makes plugins for the Helm Command Deck from
your description:

| Skill | Makes |
| --- | --- |
| `tdt-helm-widget` | A **widget**: a card on the Helm dashboard |
| `tdt-helm-tool` | A **tool**: an application in its own Helm tab, for example a kanban board |

The agent makes each plugin as an npm package in your workspace, in
`work/helm/plugins/<id>/`. It builds and validates the plugin. You then install the
plugin into Helm, and you review, approve and enable it there.

## Current availability

The skills use the Helm SDK, the `@usetdt/helm-sdk` npm package. This package is not
published to npm yet. Until it is published, `npx @usetdt/helm-sdk create` fails, and
the skills cannot make a new plugin. The stack is also not in the TDT marketplace yet.
Install it from a local checkout, as [Install the stack](#install-the-stack) describes.

## Requirements

- TDT with an initialized workspace.
- Claude Code. The stack supports Claude Code only.
- Node.js 20 or later, with npm.
- Helm, to install and use the plugins. See the Helm user guides for Helm's own
  requirements.

## Install the stack

Run these commands from the folder that contains the `tdt-helm` checkout and your
workspace. Replace `./workspace` with the path of your workspace.

```sh
tdt stack validate ./tdt-helm
tdt --workspace ./workspace stack install ./tdt-helm
tdt --workspace ./workspace stack list
```

1. `stack validate` checks the stack. If it reports a problem, do not install the stack.
2. `stack install` installs the two skills into the workspace.
3. `stack list` shows `tdt-helm`, if the installation is complete.

After the installation, start Claude Code again in your workspace. The skills are then
available as `/tdt-helm-widget` and `/tdt-helm-tool`.

The stack installs only these two skills. It adds no hooks and no brain knowledge.

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

## Change a plugin

Give the skill the id of the existing plugin, and describe the change. The agent edits
the plugin in its folder, and builds and validates it again.

You can also edit the plugin yourself. Edit only the files in `src/<id>/`. The build
makes `dist/` again. In the plugin folder, run:

```sh
npm run build      # build and validate the plugin
npm run validate   # validate without a build
npm run dev        # build again after each change in src/
```

## Install a plugin into Helm

The agent installs the plugin into Helm only if you ask. Otherwise it gives you the
command.

1. Link the plugin into Helm. Run this command in the root of the Helm repository.
   Replace the path with the absolute path of the plugin folder.

   ```sh
   npx helm plugin link /abs/path/to/workspace/work/helm/plugins/<id>
   ```

2. In Helm, open **System settings** → **Plugins**.
3. Find the plugin under **Installed**. Read its declarations: the connections, the
   calls, the writes and the capabilities.
4. If you agree, click **Approve these declarations**.
5. Turn on **Enabled in {workspace}**.

The result:

- A widget appears in the **Widget library**, in its category.
- A tool appears in the **Tools** group of the left rail, and in the tool library.

Only you can approve and enable a plugin. The agent does not do these steps.

### After a change

Helm reads a linked plugin directly from its folder. After a build, reload the widget
or tool in Helm. If the manifest changed, Helm shows **Needs review** again. Approve
the plugin again before you use it.

## What the skills do not do

- They do not change Helm, other plugins, or files outside the plugin folder.
- They do not ask for credentials, and they do not keep credentials.
- They do not approve or enable plugins in Helm.
- They do not save the plugin as brain knowledge. The plugin is your work in `work/`.

## Troubleshooting

### The SDK is not found

If `npx @usetdt/helm-sdk create` reports `E404`, the SDK is not available on npm. See
[Current availability](#current-availability).

### The build reports problems

`npm run build` validates the plugin against the Helm contracts. It reports each
problem, for example a hard-coded colour or a call that the manifest does not declare.
Ask the agent to correct the problems, or correct them in `src/<id>/`. Then build
again.

### Helm shows "Needs a newer Helm"

The plugin uses a contract version that your Helm does not support. Update Helm.

### `helm` is not found

Helm is not on npm yet, so the `helm` command is usually not on your `PATH`. Run the
command from the root of the Helm repository as `npx helm …`.

## Reference

The Helm contracts, templates and CLI are in the `@usetdt/helm-sdk` package, not in
this stack. The skills read the contracts from the SDK that the plugin installs, in
`node_modules/@usetdt/helm-sdk/docs/`.
