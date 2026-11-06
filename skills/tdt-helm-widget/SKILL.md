---
name: tdt-helm-widget
description: "Create or update a Helm Command Deck widget plugin from the user's description, using the @usetdt/helm-sdk CLI and the contracts it ships."
---

# Create a Helm widget

Use when the user asks for a Helm dashboard widget, card or panel. Needs Node 20+
with npm; installing the result needs Helm's `helm` CLI.

1. Interview briefly (chat, or TDT UI if the user said "use ui"): what it shows, where
   the data comes from (TDT tools, another MCP server, or an HTTP service), what each
   instance should be configurable by (settings), default size, and whether it writes.
   Prefer read-only. Never ask for or store credentials: HTTP services use a Helm auth
   profile the user configures in Helm.
2. Choose an id: lowercase letters, digits and dashes, starting with a letter (for
   example `user-<slug>`), not already used under `work/helm/plugins/`. To update an
   existing widget, work in its folder and skip to step 4.
3. Scaffold the plugin and install the SDK into it:
   `npx @usetdt/helm-sdk create widget work/helm/plugins/<id>`, then `npm install` in
   `work/helm/plugins/<id>/`.
4. Read the contracts before writing anything, from the plugin's installed SDK:
   `work/helm/plugins/<id>/node_modules/@usetdt/helm-sdk/docs/` — `README.md`,
   `widget.md`, `settings.md`, `connection.md`, `theme.md`, `kit.md`, `plugin.md`.
5. Edit only `src/<id>/` (`manifest.json`, `index.html`, `main.js`):
   - declare every setting (text, select, choice, checkbox, number) and connection
     operation you use; nothing else is callable;
   - style only with theme tokens and kit classes; handle unconfigured, loading, empty,
     error and ready states;
   - render connection data as text (`el()` or `textContent`), never as HTML.
6. Run `npm run build` in the plugin folder: it bundles `dist/<id>/` and validates.
   Fix every problem it reports. Report any rule you could not satisfy instead of
   bending it.
7. If the user asked you to install it, run `helm plugin link <absolute plugin path>`;
   otherwise give them that command. Then they review, approve and enable it in Helm
   (Settings → Plugins), and the widget appears in the widget library under its
   category. Approval and enabling are always the user's.

Do not edit Helm's source, other plugins or files outside the plugin folder. The plugin
folder is user-requested work under `work/`; it is not brain knowledge.
