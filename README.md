# Vade 1.0

**Vade** (`v1.0.0`) is a fast, native terminal file manager written in the [Edva](https://github.com/edva-lang/edva) systems programming language. It features an adaptive 4-pane layout, syntax-highlighted file previews, Nerd Font icons, cross-directory staging with recursive file & directory operations, and live system telemetry.

## Features

- **Adaptive 4-Pane Layout**:
  - **File Browser (Top-Left)**: Directory listing with Nerd Font file/folder icons, symlink target display (`-> target`), permissions, modification times, and human-readable sizes.
  - **Live Preview (Top-Right)**: Syntax-highlighted source code previews (via `@syntax`), directory trees, archive listings, symlink diagnostics, and color-coded binary hex dumps, plus scrollable full-screen preview mode (`F3`).
  - **History & Staging Cart (Bottom-Left)**: Pin/jump across 5 recent directory slots (`1`–`5`, `Ctrl-1`–`Ctrl-5`, or mouse click) and stage files or directories across multiple paths for batch recursive copy, move, or delete operations with interactive conflict resolution (`Overwrite`, `Newer`, `Skip`, `Auto-rename`).
  - **System Monitor (Bottom-Right)**: Live hostname, kernel, uptime, CPU load averages, RAM usage, and disk usage for the active mount.
- **Automatic Terminal Theme Detection**: Adapts to dark (**Tokyo Night**) and light (**Solarized Light**) terminal backgrounds automatically or on demand (`:theme` or `;` -> `T`).
- **Shell & Editor Integration**: Supports `cd`-on-exit (`--cwd-file`), file-picker mode (`--picker-file`), inline shell commands (`!`), and opening files in `$EDITOR` (`F4`).

## Build & Test

Ensure the sibling packages [`term`](https://github.com/edva-lang/term) and [`syntax`](https://github.com/edva-lang/syntax) are checked out alongside `vade`, then build and test with `edva`:

```bash
edva test
edva build
./vade
```

### CLI Options

```
Usage: vade [options] [directory]

Options:
  -c, --cwd-file <file>     Write last working directory to file on exit
  -p, --picker-file <file>  Pick file, write path to file, and exit
  -h, --help                Show this help message
  -V, --version             Show version information (1.0.0)
```

### Shell Wrapper (`cd` on exit)

Add this function to your `.bashrc` or `.zshrc` to change your shell's working directory when exiting `vade`:

```sh
v() {
  local tmp="$(mktemp -t vade-cwd.XXXXXX)"
  vade --cwd-file="$tmp" "$@"
  if [ -f "$tmp" ]; then
    local dir="$(cat "$tmp")"
    rm -f "$tmp"
    [ -d "$dir" ] && [ "$dir" != "$PWD" ] && cd "$dir"
  fi
}
```

## Keybindings

| Key | Action |
| :--- | :--- |
| `↑` / `↓`, `PgUp` / `PgDn`, `Home` / `End` | Navigate file list (or scroll preview when in `F3` full-screen mode) |
| `Enter` / `→` | Enter directory or open file |
| `Backspace` / `←` | Navigate up to parent directory |
| `a`–`z` | Jump to visible row labeled `a`–`z` (or cycle through matching rows) |
| `Space` / `Ins` | Stage / unstage file or directory in cart (cross-directory) |
| `1`–`5` | Fast jump to recent directory slot (or click slot in History pane) |
| `Ctrl-1`–`Ctrl-5` | Lock / unlock (pin) history directory slot |
| `~` / `-` | Jump to Home (`~`) or previous directory (`-`) |
| `>` | Prompt to change directory directly (`cd`) |
| `/` | Live-filter files in the current directory |
| `;` | Sort (`Name`, `Size`, `Mtime`, `Ext`, `Reverse`), View (`Hidden`, `Glyphs`, `Theme`), and Selection (`All`, `Invert`, `Clear`) |
| `!` | Run shell command in current directory (`Tab` / `Ctrl-Enter` inserts hovered filename) |
| `:` | Command prompt (`:cd <dir>`, `:mkdir <dir>`, `:touch <file>`, `:sort <name\|size\|time\|ext\|rev>`, `:icons`, `:hidden`, `:all`, `:clear`, `:theme`, `:dark`, `:light`, `:q`) |
| `Shift-↑` / `Shift-↓` | Scroll preview pane |
| `Ctrl-H` / `F1` / `?` | Toggle Help modal |
| `F2` | Rename hovered file or directory |
| `F3` | Toggle full-screen preview |
| `F4` | Edit hovered file in `$EDITOR` |
| `F5` / `F6` | Copy (`F5`) or Move (`F6`) staged items to current directory (with conflict resolution) |
| `F7` | Create new file or directory (trailing `/` creates a directory) |
| `F8` / `Del` | Recursively delete hovered or staged items |
| `F10` / `Esc Esc` | Quit `vade` |
