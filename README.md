# tinted-claude-code

[Tinted Theming](https://github.com/tinted-theming) template for
[Claude Code](https://docs.claude.com/en/docs/claude-code) custom themes.

It compiles any Base16, Base24, or Tinted8 scheme into a Claude Code
custom-theme JSON file, so you can theme Claude Code from the same scheme you
use for your terminal, editor, and everything else — and switch them all at
once with [Tinty](https://github.com/tinted-theming/tinty).

## What it produces

For every scheme, the builder renders **two flavors** of the same theme:

| Flavor | Output | What it is | Use when |
| ------ | ------ | ---------- | -------- |
| **Static** | [`themes/`](./themes)`/{system}-{slug}.json` | A ready-to-use Claude Code theme JSON. | You want a plain file to copy, or you don't have Node. |
| **Executable** | [`scripts/`](./scripts)`/{system}-{slug}.js` | A Node script that prints the theme JSON to stdout. | You want **true shimmer** and **themed diffs** (computed via color math). Requires Node. |

A theme file is a Claude Code custom theme — a JSON document with a `name`, a
`base` preset (`dark`/`light`), and an `overrides` map of color tokens:

```json
{
  "name": "Ayu Dark",
  "base": "dark",
  "overrides": {
    "text": "#e6e1cf",
    "claude": "#59c2ff",
    "error": "#f07178",
    "success": "#aad94c",
    "...": "..."
  }
}
```

Claude Code custom themes require **Claude Code v2.1.118 or later**. Tokens it
doesn't recognize and invalid color values are silently ignored, so the themes
stay forward/backward compatible.

### Why two flavors?

A few Claude Code tokens need *computed* colors, not direct palette slots:

- **Shimmer** (`claudeShimmer`, `warningShimmer`, the `rainbow_*_shimmer` set, …)
  is a lighter/darker twin of its paired color used in the animated spinner
  gradient. Base16/Base24 have no lighter-accent slot to point at.
- **Diff backgrounds** (`diffAdded`, `diffRemoved`, their dimmed and word-level
  variants) are low-saturation tints of green/red blended into the editor
  background.

The Tinted builder uses logic-less Mustache (pure substitution — no color math),
so the **static JSON** can't compute these: its shimmer tokens reuse their
paired color and its diff tokens are omitted so they inherit Claude Code's tuned
out-of-the-box contrast.

The **executable JS** flavor does the math at apply time. It also decides
`dark` vs `light` by measuring the **CIE L\*** (perceptual lightness) of the
foreground vs the background rather than trusting the scheme's declared
`variant`, so mislabeled schemes still render correctly.

## Usage with Tinty

Add an item to your Tinty `config.toml`
(`~/.config/tinted-theming/tinty/config.toml`).

**Executable flavor (recommended — true shimmer + themed diffs, needs Node):**

```toml
[[items]]
name = "tinted-claude-code"
path = "https://github.com/bezhermoso/tinted-claude-code"
themes-dir = "scripts"
theme-file-extension = ".js"
supported-systems = ["base16", "base24", "tinted8"]
hook = "mkdir -p \"$HOME/.claude/themes\" && node \"$TINTY_THEME_FILE_PATH\" > \"$HOME/.claude/themes/tinty.json\""
```

**Static flavor (no Node, plain file copy):**

```toml
[[items]]
name = "tinted-claude-code"
path = "https://github.com/bezhermoso/tinted-claude-code"
themes-dir = "themes"
theme-file-extension = ".json"
supported-systems = ["base16", "base24", "tinted8"]
hook = "mkdir -p \"$HOME/.claude/themes\" && cp -f \"$TINTY_THEME_FILE_PATH\" \"$HOME/.claude/themes/tinty.json\""
```

Then:

```sh
tinty install            # clone this repo + build the theme files
tinty apply base16-ayu-dark
```

The hook writes the matching theme to `~/.claude/themes/tinty.json` (the static
flavor copies the file; the executable flavor runs it through Node). **The first
time only**, open Claude Code and run `/theme`, then pick the **Tinty** entry
(Claude stores this as `theme: "custom:tinty"` in `~/.claude/settings.json`).

From then on, every `tinty apply` overwrites `tinty.json`. Claude Code watches
`~/.claude/themes/` and **hot-reloads**, so the running session re-themes
instantly — no restart, no re-selecting. The theme's display `name` updates to
the current scheme while the `custom:tinty` selection stays valid (selection is
keyed off the file name, not the `name` field).

### Why a single fixed `tinty.json`?

Claude Code selects a theme by file slug. Pinning Tinty's output to one slug
(`tinty`) means you select it once and Tinty drives it forever. If you'd rather
keep a per-scheme picker entry instead, drop `tinty.json` from the hook and copy
to the scheme's own name:

```toml
hook = "mkdir -p \"$HOME/.claude/themes\" && cp -f \"$TINTY_THEME_FILE_PATH\" \"$HOME/.claude/themes/$TINTY_SCHEME_ID.json\""
```

…but then you must re-select the theme in `/theme` each time you switch.

## Manual install (without Tinty)

Static flavor — copy a file straight in:

```sh
mkdir -p ~/.claude/themes
cp themes/base16-ayu-dark.json ~/.claude/themes/
```

Executable flavor — run the script into your themes dir:

```sh
mkdir -p ~/.claude/themes
node scripts/base16-ayu-dark.js > ~/.claude/themes/ayu-dark.json
```

Run `/theme` in Claude Code and select it.

## Supported systems

| System   | Source tokens                              | Notes |
| -------- | ------------------------------------------ | ----- |
| Base16   | `base00`–`base0F`                          | |
| Base24   | `base00`–`base17`                          | Rendered by the same template as Base16 (uses the shared `base00`–`base0F` slots). |
| Tinted8  | `palette.*`, `ui.*` (semantic)             | Richer mapping — uses the scheme's semantic `ui.*` roles. |

### Token mapping (highlights)

The full mapping lives in [`templates/`](./templates). A few of the choices:

| Claude Code token            | Base16 / Base24      | Tinted8                          |
| ---------------------------- | -------------------- | -------------------------------- |
| `text`                       | `base05`             | `ui.global.foreground.normal`    |
| `claude` (brand accent)      | `base0D`             | `ui.accent.normal`               |
| `error`                      | `base08`             | `ui.status.error`                |
| `success`                    | `base0B`             | `ui.status.success`              |
| `warning`                    | `base0A`             | `ui.status.warning`              |
| `planMode`                   | `base0D`             | `palette.blue.normal`            |
| `autoAccept`                 | `base0B`             | `palette.green.normal`           |
| message/selection backgrounds| `base01`/`base02`    | `ui.global.background.dark`, `ui.selection.background` |
| `*_FOR_SUBAGENTS_ONLY` (8)   | `base08`–`base0F`    | `palette.*`                      |
| `rainbow_*` (7)              | `base08`–`base0E`    | `palette.*`                      |

**Shimmer** (`*Shimmer`, `rainbow_*_shimmer`) and **diff backgrounds**
(`diffAdded`/`diffRemoved` + dimmed + word-level) are handled per flavor:

- **Static JSON** — shimmer reuses its paired color; diff tokens are omitted and
  inherit Claude Code's tuned `dark`/`light` base preset.
- **Executable JS** — shimmer is computed by blending each paired color toward
  white (dark themes) or black (light themes); diff backgrounds are computed by
  blending green/red into the background (subtle for lines, fainter for context,
  stronger for word-level). Blend factors are tunable constants at the top of
  each `templates/*.js.mustache`.

## Building locally

Requires
[`tinted-builder-rust`](https://github.com/tinted-theming/tinted-builder-rust):

```sh
# Build against a local schemes checkout…
tinted-builder-rust build . --schemes-dir /path/to/tinted-theming/schemes

# …or fetch the latest schemes first
tinted-builder-rust build . --sync
```

The generated `themes/` (static JSON) and `scripts/` (executable JS) directories
are committed to the repo so it works without a build step. CI
([`.github/workflows/update.yml`](./.github/workflows/update.yml)) rebuilds
weekly against the latest upstream schemes via the shared `tinted-theming/home`
workflow.

## License

[MIT](./LICENSE)
