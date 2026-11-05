---
name: tdt-helm-tool
description: "Create or update a Helm tool (a tabbed app such as a kanban board) from the user's description, following the installed Helm tool, settings, connection and theme contracts."
---

# Create a Helm tool

Use when the user wants a full-tab tool in Helm rather than a dashboard card.

1. Read `.tdt/stacks/tdt-helm/docs/tool.md`, `settings.md`, `connection.md`, `theme.md`.
2. Interview briefly: the job the tool does, its data sources, per-instance settings,
   whether several instances make sense (`multi`), and which host capabilities it needs
   (`orchestrate` to start builds, `terminal`, `chat`). Request only what is needed.
3. Choose an id `user-<slug>` that is not already used under `work/helm/tools/`.
4. Copy `.tdt/stacks/tdt-helm/templates/tool/` to `work/helm/tools/<id>/` and fill
   `manifest.json` and `index.html`, using the standard tool header, token-only
   styling, 140px bottom padding for the chat dock, and all empty/error states.
5. For orchestration, start builds only through `helm.orchestrate.build`, which runs
   `tdt-software-production-build`. Never mark tasks done; the build skill and the
   user own task status.
6. Validate against the contract and tell the user where the tool appears (left rail →
   Tools, and the tool library).
