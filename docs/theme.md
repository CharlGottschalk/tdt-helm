# Theme contract v1

Extensions must look native in every Helm theme (HUD, Aurora Glass, Neon Cyberpunk,
Monolith, Satin Slate and any added later). They do that by using only the tokens
below. The source of truth is `src/styles/tokens.css` and `src/styles/themes.css`.

## Tokens extensions may use

| Group | Tokens |
| --- | --- |
| Text | `--c-text-0` (primary) · `--c-text-1` (body) · `--c-text-2` (muted) · `--c-text-3` (faint, labels) · `--c-on-accent` (text on accent fills) |
| Surfaces | `--c-surface` · `--c-surface-raised` · `--c-surface-hover` · `--c-surface-active` · `--c-well` (inputs) · `--c-chrome` (overlays) · `--c-bg-0` |
| Lines | `--c-line` · `--c-line-strong` · `--c-line-hot` · `--c-grid` |
| Accent | `--c-cyan` (primary accent, whatever its hue) · `--c-cyan-ghost` · `--c-accent-2` (frame accent) · `--c-accent-hover` |
| Signal | `--c-red` / `-ghost` critical · `--c-amber` / `-ghost` attention · `--c-green` / `-ghost` nominal · `--c-violet` / `-ghost` concept · `--c-blue` / `-ghost` document |
| Alpha tints | `rgb(var(--rgb-accent) / α)` and `rgb(var(--rgb-red|amber|green|violet|blue) / α)` |
| Type | `--font-display` · `--font-body` · `--font-mono` · `--fs-2xs` … `--fs-3xl` · `--tracking-hud` · `--tracking-title` · `--display-weight` |
| Space | `--sp-1` (4) … `--sp-10` (40) |
| Shape | `--radius` · `--radius-sm` · `--clip-cut-sm|md|lg` · `--shape-identity` · `--skew` |
| Effects | `--glow-cyan` · `--glow-accent-2` · `--glow-text` · `--shadow-float` · `--blur-panel` |
| Motion | `--dur-fast` (120ms) · `--dur-med` (220ms) · `--dur-slow` (420ms) · `--ease-out` |

Sandboxed extensions receive the resolved values of all of these as CSS variables on
`:root`, re-sent on every theme change. Global helper classes are injected too:
`.hud-label`, `.mono`, `.display`.

## Rules

1. **No literal colours, fonts, radii or shadows.** If you need a token that isn't
   listed, propose it for the contract; don't hard-code.
2. **Signal colours carry meaning only:** red critical, amber attention, green
   nominal/done, violet concept, blue document. Don't use them for decoration.
3. **Accent sparingly:** one primary (`--c-cyan` fill) action per widget body.
4. **Contrast:** body text uses `--c-text-0`/`-1`; `--c-text-2`/`-3` are for labels
   and metadata only. All text tokens meet WCAG AA in every shipped theme on
   `--c-surface`; don't put text on signal fills except `--c-on-accent` on `--c-cyan`.
5. **Respect motion settings:** wrap non-essential animation in
   `@media (prefers-reduced-motion: no-preference)`; Helm also sets
   `html[data-motion='reduced']`.
6. **Glows and shapes are the theme's job.** Use `--glow-*`, `--radius*` and
   `--clip-cut-*` rather than inventing effects: Satin Slate and Monolith turn glows
   off, Glass rounds corners, Cyberpunk cuts them.
