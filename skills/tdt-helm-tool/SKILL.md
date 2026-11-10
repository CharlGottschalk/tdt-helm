---
name: tdt-helm-tool
description: "Create or update a Helm tool plugin (a tabbed app such as a kanban board) from the user's description, using the @usetdt/helm-sdk CLI and the contracts it ships."
---

# Create a Helm tool

Use when the user wants a full-tab tool in Helm rather than a dashboard card. Needs
Node 20+ with npm; installing the result needs Helm's `helm` CLI.

1. Interview briefly (chat, or TDT UI if the user said "use ui"): the job the tool does,
   its data sources, per-instance settings, whether several instances make sense
   (`multi`), and which host capabilities it needs (`orchestrate` to start builds,
   `terminal`, `chat`). Request only what is needed. Never ask for or store credentials.
2. Choose an id: lowercase letters, digits and dashes, starting with a letter (for
   example `user-<slug>`), not already used under `work/helm/plugins/`. To update an
   existing tool, work in its folder and skip to step 4.
3. Scaffold the plugin and install the SDK into it:
   `npx @usetdt/helm-sdk create tool work/helm/plugins/<id>`, then `npm install` in
   `work/helm/plugins/<id>/`.
4. Read the contracts before writing anything, from the plugin's installed SDK:
   `work/helm/plugins/<id>/node_modules/@usetdt/helm-sdk/docs/` — `README.md`,
   `tool.md`, `settings.md`, `connection.md`, `theme.md`, `kit.md`, `plugin.md`.
5. Edit only `src/<id>/` (`manifest.json`, `index.html`, `main.js`), keeping the
   standard tool header, token-only styling, the 140px bottom padding for the chat dock,
   and every unconfigured, loading, empty and error state. Render connection data as
   text, never as HTML.
6. For orchestration, start builds only through `helm.orchestrate.build`, which runs
   `tdt-software-production-build`. Never mark tasks done; the build skill and the
   user own task status.
7. Run `npm run build` in the plugin folder: it bundles `dist/<id>/` and validates.
   Fix every problem it reports. Report any rule you could not satisfy instead of
   bending it.
8. If the user asked you to install it, run `helm plugin link <absolute plugin path>`;
   otherwise give them that command. Then they review, approve and enable it in Helm
   (Settings → Plugins), and the tool appears under Tools in the left rail and in the
   tool library. Approval and enabling are always the user's.

Do not edit Helm's source, other plugins or files outside the plugin folder. The plugin
folder is user-requested work under `work/`; it is not brain knowledge.
