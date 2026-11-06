# UI kit v1

Helm injects these classes into every sandboxed frame, after the theme tokens
([theme.md](theme.md)), so plugins match Helm in all five themes without shipping
their own CSS. Source: `KIT_CSS` in `@usetdt/helm-sdk`.

| Class | Use |
| --- | --- |
| `h-stack` | Vertical stack with `--sp-2` gaps |
| `h-muted` | Secondary text |
| `h-value` | A number or headline value in the display font |
| `h-state` | Replaces content for a non-ready state; add `h-state--loading`, `h-state--error`. Plain `h-state` is the empty/unconfigured state |
| `h-list` + `h-row` | A list of rows: label left, value right, divider between. A `button.h-row` is a clickable row |
| `h-badge` | Small status label; tones `h-badge--cyan`, `--amber`, `--red`, `--green`, `--violet`, `--blue` |
| `h-btn`, `h-btn--ghost` | Primary and secondary buttons |
| `h-field` + `h-input` | Labelled form control (`<label class="h-field">Name <input class="h-input"></label>`) |

The kit also sets `box-sizing: border-box` and a visible `:focus-visible` outline.

Every widget handles five states: unconfigured (`h-state` with an
`h-btn` that calls `helm.openSettings()`), loading, empty, error (with a Retry
`h-btn--ghost`) and ready.

```html
<div id="app" class="h-state h-state--loading">Loading…</div>
<script>
  // ready
  app.className = ''
  app.replaceChildren(list)   // <ul class="h-list"><li class="h-row"><span>Projects</span><b class="h-value">8</b></li></ul>
</script>
```

Badge text uses `--c-text-0` on the tone's ghost colour, so contrast holds in every
theme; the tone colour is the border. Don't put text in a signal colour on a
ghost background.
