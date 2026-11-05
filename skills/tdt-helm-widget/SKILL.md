---
name: tdt-helm-widget
description: "Create or update a Helm Command Deck widget from the user's description, following the installed Helm widget, settings, connection and theme contracts."
---

# Create a Helm widget

Use when the user asks Helm (or you) to make a dashboard widget, card or panel.

1. Read the installed contracts before writing anything:
   `.tdt/stacks/tdt-helm/docs/widget.md`, `settings.md`, `connection.md`, `theme.md`.
2. Interview briefly (chat, or TDT UI if the user said "use ui"): what it shows, where
   the data comes from (TDT tools, another MCP server, or an HTTP service), what each
   instance should be configurable by (settings), default size, and whether it writes.
   Prefer read-only. Never ask for or store credentials: HTTP services use a Helm auth
   profile the user configures in Helm.
3. Choose an id `user-<slug>` that is not already used under `work/helm/widgets/`.
4. Copy `.tdt/stacks/tdt-helm/templates/widget/` to `work/helm/widgets/<id>/` and fill
   `manifest.json` and `index.html`:
   - declare every setting (text, select, choice, checkbox, number) and connection
     operation you use; nothing else is callable;
   - style only with theme tokens; handle unconfigured, loading, empty, error and ready states;
   - render untrusted data with `textContent`.
5. Validate the manifest against the contract rules (ids, sizes, settings limits, no
   secret-like setting ids). Report any rule you could not satisfy instead of bending it.
6. Tell the user the widget is in Helm's widget library under its category, and how to
   configure it. Do not edit Helm's source or other widgets.

Saving these files is user-requested work under `work/`; it is not brain knowledge.
