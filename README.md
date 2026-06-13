# tinted-claude-code

[Tinted Theming](https://github.com/tinted-theming) template for
[Claude Code](https://docs.claude.com/en/docs/claude-code) custom themes.

It compiles any Base16, Base24, or Tinted8 scheme into a Claude Code
custom-theme JSON file, so you can theme Claude Code from the same scheme you
use for your terminal, editor, and everything else — and switch them all at
once with [Tinty](https://github.com/tinted-theming/tinty).

## What it produces

For every scheme, the builder renders one Claude Code theme file into
[`themes/`](./themes), named `{system}-{slug}.json`:

```
themes/base16-ayu-dark.json
themes/base24-catppuccin-mocha.json
themes/tinted8-gruvbox-dark.json
```

Each file is a Claude Code custom theme — a JSON document with a `name`, a
`base` preset (`dark`/`light`, taken from the scheme variant), and an
`overrides` map of color tokens:

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

## Usage with Tinty

Add an item to your Tinty `config.toml`
(`~/.config/tinted-theming/tinty/config.toml`):

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

The hook copies the matching theme into `~/.claude/themes/tinty.json`. **The
first time only**, open Claude Code and run `/theme`, then pick the **Tinty**
entry (Claude stores this as `theme: "custom:tinty"` in
`~/.claude/settings.json`).

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

Pick any file from [`themes/`](./themes) and drop it into your themes dir:

```sh
mkdir -p ~/.claude/themes
cp themes/base16-ayu-dark.json ~/.claude/themes/
```

Run `/theme` in Claude Code and select it.

## Supported systems

| System   | Source tokens                              | Notes |
| -------- | ------------------------------------------ | ----- |
| Base16   | `base00`–`base0F`                          | |
| Base24   | `base00`–`base17`                          | Rendered by the same template as Base16 (uses the shared `base00`–`base0F` slots). |
| Tinted8  | `palette.*`, `ui.*` (semantic)             | Richer mapping — uses the scheme's semantic `ui.*` roles and `bright` palette variants for shimmer/accent colors. |

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
| `rainbow_*` + shimmers (14)  | `base08`–`base0E`    | `palette.*` (`bright` for shimmer)|

**Diff backgrounds** (`diffAdded`, `diffRemoved`, …) are intentionally *not*
overridden: they require blended, low-saturation background tints that can't be
derived from a fixed palette without color math, so they inherit from Claude
Code's `dark`/`light` base preset, which is tuned to read well.

## Building locally

Requires
[`tinted-builder-rust`](https://github.com/tinted-theming/tinted-builder-rust):

```sh
# Build against a local schemes checkout…
tinted-builder-rust build . --schemes-dir /path/to/tinted-theming/schemes

# …or fetch the latest schemes first
tinted-builder-rust build . --sync
```

The generated `themes/` directory is committed to the repo so it works without a
build step. CI ([`.github/workflows/update.yml`](./.github/workflows/update.yml))
rebuilds weekly against the latest upstream schemes via the shared
`tinted-theming/home` workflow.

## License

[MIT](./LICENSE)
