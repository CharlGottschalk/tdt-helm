---
name: tdt-helm-theme
description: "Create or update a Helm theme plugin (token values only) from the user's description of a look, using the @usetdt/helm-sdk CLI and the theme contract it ships."
---

# Create a Helm theme

Use when the user asks for a Helm theme, colour scheme or look. A theme is token
values only: no code, no calls, no images. Needs Node 20+ with npm and
`@usetdt/helm-sdk` 0.2.0 or later; installing the result needs Helm's `helm` CLI.

1. Interview briefly (chat, or TDT UI if the user said "use ui"): the look in a few
   words, the accent and second accent, the grounds (background and panel colours),
   light or dark, corners (sharp or round) and fonts, and whether the rails (sidebars,
   tab bar, chat dock) should differ from the content, as Monolith has dark rails.
   Use the user's words for the name and tagline.
2. Choose an id: lowercase letters, digits and dashes, starting with a letter, 2–64
   characters, not already used under `work/helm/plugins/` and not a built-in theme
   (`hud`, `glass`, `cyber`, `mono`, `satin`). To update an existing theme, work in its
   folder and skip to step 4.
3. Scaffold the plugin and install the SDK into it:
   `npx @usetdt/helm-sdk create theme work/helm/plugins/<id>`, then `npm install` in
   `work/helm/plugins/<id>/`.
4. Read the contracts before writing anything, from the plugin's installed SDK:
   `work/helm/plugins/<id>/node_modules/@usetdt/helm-sdk/docs/theme.md` (Theme plugins:
   the tokens a theme can set, their value forms, and contrast) and `plugin.md`.
5. Edit only `themes/<id>/`:
   - `manifest.json`: `name` (at most 60 characters) and `tagline` (at most 140);
   - `tokens.css`: only `--token: value;` lines and `/* comments */`. Set the hues
     first (`--rgb-accent`, `--rgb-accent-2`, `--rgb-line`, the surfaces), then the
     grounds and the four text colours, `--c-on-accent` and `--c-accent-hover`. Set a
     derived token only to tune it;
   - for different rails, add `chrome.css` with the rail tokens and set
     `"chrome": "chrome.css"` in the manifest.
   No selectors, braces, at-rules, `url()`, `var()` or `!important`. Fonts must be
   fonts that Helm bundles or system fonts; a theme cannot load font files.
6. Run `npm run validate` in the plugin folder. Fix every error. Report each contrast
   warning plainly (the pair and its ratio) and offer a fix that keeps the look, for
   example a lighter text colour. Below AA (4.5:1) is allowed, but Helm flags it with
   **Below AA contrast**; keep it only if the user chooses to.
7. If the user asked you to install it, run `helm plugin link <absolute plugin path>`;
   otherwise give them that command. Then they review and approve it in Helm
   (System settings → Plugins), and select it in System settings → Appearance → Theme.
   Themes have no per-workspace enable: an approved theme is available in every
   workspace. A changed token file needs a new approval. Approval is always the
   user's.

Do not edit Helm's source, other plugins or files outside the plugin folder. The plugin
folder is user-requested work under `work/`; it is not brain knowledge.
