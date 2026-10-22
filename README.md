# Vade

**Vade** is a fast, native terminal file manager written in the [Edva](https://github.com/edva-lang/edva) systems programming language. It features an adaptive 4-pane layout, syntax-highlighted file previews, Nerd Font icons, cross-directory staging, and live system telemetry.

## Features

- **Adaptive 4-Pane Layout**:
  - **File Browser (Top-Left)**: Directory listing with Nerd Font file/folder icons, permissions, modification times, and human-readable sizes.
  - **Live Preview (Top-Right)**: Syntax-highlighted source code previews (via `@syntax`), directory contents, archive listings, and binary detection, plus full-screen preview mode (`F3`).
  - **History & Staging Cart (Bottom-Left)**: Pin/jump across 5 recent directory slots (`1`–`5`, `Ctrl-1`–`Ctrl-5`) and stage items across multiple directories for batch copy, move, or delete operations.
  - **System Monitor (Bottom-Right)**: Live hostname, kernel, uptime, CPU load averages, RAM usage, and disk usage for the active mount.
- **Automatic Terminal Theme Detection**: Adapts to dark (**Tokyo Night**) and light (**Solarized Light**) terminal backgrounds automatically or on demand (`:theme`).
- **Shell & Editor Integration**: Supports `cd`-on-exit (`--cwd-file`), file-picker mode (`--picker-file`), inline shell commands (`!`), and opening files in `$EDITOR` (`F4`).

## Build & Run

Ensure the sibling packages [`term`](https://github.com/edva-lang/term) and [`syntax`](https://github.com/edva-lang/syntax) are checked out alongside `vade`, then build with `edva`:

```bash
edva pkg build
./vade
```

### CLI Options

```
Usage: vade [options] [directory]

Options:
  -c, --cwd-file <file>     Write last working directory to file on exit
  -p, --picker-file <file>  Pick file, write path to file, and exit
  -h, --help                Show this help message
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
| `↑` / `↓`, `PgUp` / `PgDn`, `Home` / `End` | Navigate file list (or mouse wheel) |
| `Enter` | Enter directory or open file |
| `Backspace` | Navigate up to parent directory |
| `a`–`z` | Jump / cycle through files starting with letter |
| `Space` / `Ins` | Stage / unstage file or directory in cart (cross-directory) |
| `1`–`5` | Fast jump to recent directory slot |
| `Ctrl-1`–`Ctrl-5` | Lock / unlock (pin) history directory slot |
| `~` / `-` | Jump to Home (`~`) or previous directory (`-`) |
| `>` | Prompt to change directory directly (`cd`) |
| `/` | Live-filter files in the current directory |
| `;` | Sort (`Name`, `Size`, `Mtime`, `Ext`, `Reverse`), `Hidden` toggle, and selection menu |
| `!` | Run shell command in current directory (`Tab` / `Ctrl-Enter` inserts hovered filename) |
| `:` | Command prompt (`:cd <dir>`, `:mkdir <dir>`, `:touch <file>`, `:theme`, `:dark`, `:light`, `:q`) |
| `Shift-↑` / `Shift-↓` | Scroll preview pane |
| `Ctrl-H` / `F1` / `?` | Toggle Help modal |
| `F2` | Rename hovered file or directory |
| `F3` | Toggle full-screen preview |
| `F4` | Edit hovered file in `$EDITOR` |
| `F5` / `F6` | Copy (`F5`) or Move (`F6`) staged items to current directory (with conflict resolution) |
| `F7` | Create new file or directory (trailing `/` creates a directory) |
| `F8` / `Del` | Delete hovered or staged items |
| `F10` / `Esc Esc` | Quit `vade` |
