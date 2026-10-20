# vade — Terminal File Manager Specification

`vade` is a high-performance, single-pane modal terminal file manager written natively in **Edva**, inspired by modern TUI managers like Yazi, Ranger, and nnn, but tailored for speed, ergonomics, and seamless keyboard navigation.

---

## 1. Core Architecture & Design Principles

- **Language & Runtime**: Native Edva, using standard library modules (`std/sys`, `std/libc`, `std/io`, `std/str`, `std/color`, `std/ccl`, `std/uring`, `std/fmt`, `std/sort`).
- **Location**: Standalone project at `/home/ike/Z/vade`.
- **Zero Flickering**: Double-buffered terminal cell matrix with dirty-diff rendering over alternate screen buffer (`\e[?1049h`).
- **Zero I/O Stuttering**: Asynchronous background task queue powered by `io_uring` / chunked I/O for directory reads, stat scanning, and file operations.
- **Fast-Access Ergonomics**: Onscreen visual letter keys (`a`–`z`) for direct 1-key jumping and double-tap opening.

---

## 2. Layout & View Modes

The UI uses a single pane maximizing screen utilization, framed by a top path/header bar and a bottom status/progress bar. Users switch the active view mode instantly via number keys **1–4**:

```
+-- [/home/ike/projects/vade] ------------------------- 12 items, 2 selected (42.1 MB) --+
| a  README.md         4.2 KB  -rw-r--r--  2026-10-15 20:30                              |
| b* src/              DIR     drwxr-xr-x  2026-10-15 20:45                              |
| c* build.dva         1.1 KB  -rw-r--r--  2026-10-15 20:40                              |
| d  config.ccl        820 B   -rw-r--r--  2026-10-15 20:25                              |
| e  LICENSE           1.0 KB  -rw-r--r--  2026-10-15 20:10                              |
+----------------------------------------------------------------------------------------+
| [Copying 2 files -> /tmp] [=======>    ] 65% (12.4 MB/s)  | [1:Long 2:Short 3:Tree 4:Prev] |
+----------------------------------------------------------------------------------------+
```

### View Modes
1. **`1` — Long List View**:
   - Detailed columnar display: fast-access letter tag, selection mark (`*`), filename, formatted file size, permissions (`rwxr-xr-x`), modification date/time.
   - Colored by file type (executable, directory, archive, symlink, code, media).
2. **`2` — Short List View**:
   - Multi-column compact grid of file names with fast-access letters.
   - Maximizes the number of entries visible simultaneously without scrolling.
3. **`3` — Tree View**:
   - Hierarchical collapsible directory tree with tree branches (`├──`, `└──`).
   - Inline expansion/collapse of folders to quickly inspect nested structures.
4. **`4` — Preview View**:
   - Full-pane dedicated inspector of the currently selected file or directory.

---

## 3. Navigation & Interaction Model

### Fast-Access Letter Navigation (`a`–`z`)
- Every visible entry on screen is tagged with a distinct lowercase letter (`a`, `b`, `c`, ... `z`).
- **Single Press**: Pressing the letter instantly moves the cursor to that file/folder.
- **Double Tap / Enter**: Double-tapping the letter (or pressing `Enter`) enters the directory or opens the file.
- Standard arrow keys (`Up`, `Down`, `Left` for parent directory, `Right` to enter), `PageUp`, `PageDown`, `Home`, and `End` are always available.

### Inline Substring Filtering
- Press **`/`** to open the inline filter prompt at the bottom bar.
- Type any substring to filter visible entries in real time.
- Matching entries are dynamically reassigned `a`–`z` fast-access letters for immediate jumping.
- Press **`Esc`** or **`Enter`** to exit filter mode; `Esc` clears the filter and returns to the full directory listing.

### History Navigation & Pinned Slots (`1`–`5`, `Ctrl-1`–`Ctrl-5`)
- Press **`1`–`5`** to instantly navigate to any of the 5 visited directory history slots in the bottom-left pane.
- Press **`Ctrl-1`–`Ctrl-5`** to toggle locking (pinning) the respective slot in place. If the requested slot is empty, a warning is displayed and no action is taken.
- **Excluded from Rotation**: Pinned directories are rendered with a distinct reddish highlight, and are never overwritten or expelled when browsing new directories; only unlocked slots rotate.

---

## 4. Selection & File Operations

### Global Cross-Directory Selection Accumulator
- Selection persists seamlessly across directory transitions: items selected in one folder remain staged when navigating to other folders.
- **`Space` / `Insert`**: Toggle selection mark on the current entry and advance cursor.
- **`A`**: Select all entries in the active directory view.
- **`C`**: Clear entire selection cart across all directories.
- Staged cart items appear in the bottom-left pane with total item count and formatted aggregate size.

### Conflict Resolution Policies
When pasting items (`P`, `F5`, `F6`) into a destination directory where files of the same name already exist, `vade` prompts for conflict resolution:
`Conflict: <filename> exists. [O]verwrite [N]ewer [S]kip [A]uto-rename [C]ancel:`
- **`[O]verwrite`**: Overwrite existing destination file with the source file.
- **`[N]ewer`**: Overwrite only if the source file has a strictly newer modification time (`mtime`).
- **`[S]kip`**: Do not copy/move this item; proceed with remaining items in the queue.
- **`[A]uto-rename`**: Preserve both files by saving the new item with an enumerated suffix (e.g., `file (1).txt`).
- **`[C]ancel` / `Esc`**: Abort the entire remaining transfer operation safely.

### File Manipulation Keybindings
All action keys use uppercase letters or special keys:
- **`Y` (Yank / Copy)**: Stage selected items (or hovered item if selection is empty) for copy.
- **`X` (Cut / Move)**: Stage selected items (or hovered item if selection is empty) for move.
- **`P` (Paste)**: Paste staged items into the active directory with conflict collision checking.
- **`D` / `F8`**: Delete selected items (or hovered item) permanently with confirmation dialog.
- **`F2` / `R`**: Prompt to rename hovered entry.
- **`F7` / `N`**: Create new file or directory (names ending in `/` create directories).
- **`!`**: Open shell command prompt (interactive command with file path interpolation).
- **`Ctrl-H` / `F1`**: Toggle help modal keybindings overlay.

### CLI Integration: Cd-on-Exit & File Picker Mode
- **`-c, --cwd-file <path>`**: On exit (via `Q`, `:q`, `F10`), write the last visited working directory to `<path>`. Shell wrapper functions (e.g. `yy` or `v`) can automatically `cd` the calling shell to this path.
- **`-p, --picker-file <path>`**: File picker mode. Pressing `Enter` on a regular file (or selecting multiple items and pressing `Enter`/quitting) writes the selected absolute paths separated by newlines to `<path>` and exits immediately.
- **Startup Hint**: Upon startup, displays `"Ctrl-H for shortcuts"` in the status bar, automatically dismissed upon the first keypress.

---

## 5. Preview Engine

When in Preview View (`4`) or hovering in preview mode:
1. **Source Code & Text Files**:
   - Built-in syntax highlighting (keywords, strings, comments, numbers) for common languages (Edva, C, Rust, Python, Go, Shell, Markdown, JSON, CCL, HTML/CSS).
   - Line numbers with scroll offset.
2. **Binary Files**:
   - Built-in hex viewer displaying hexadecimal bytes and ASCII representation columns.
3. **Directories**:
   - Directory statistics: total number of files/subdirectories, total recursive size, and mini tree summary.
4. **External Preview Command Fallback**:
   - If an external tool is configured for a filetype (e.g. `bat`, `mediainfo`, `cat`, or image thumbnailers), `vade` invokes the hook and displays its output.

---

## 6. Terminal Engine & Rendering

Self-contained native Edva terminal subsystem:
- **Raw Mode**: Sets `termios` flags (`tcgetattr`/`tcsetattr`) via libc/sys FFI.
- **Alternate Screen Buffer**: Uses ANSI sequences `\e[?1049h` / `\e[?1049l` to preserve the user's terminal shell history upon exit.
- **Double Buffer Cell Grid**:
  - Two screen buffers (back and front matrix of cells: character, foreground color, background color, style attributes).
  - Dirty diff algorithm: only changed cells or scanlines are emitted to stdout, minimizing terminal write overhead and eliminating flicker.
- **TrueColor & 256 Colors**: Full 24-bit color support integrated with `std/color.dva`.
- **Resize Handling**: Traps `SIGWINCH` and queries `TIOCGWINSZ` ioctl to adaptively recalculate layout and redraw immediately.
- **Mouse Support**: Optional mouse click selection and wheel scrolling (`\e[?1000h`, `\e[?1006h`).

---

## 7. Configuration & Theming (CCL)

Configuration is specified in standard Edva **CCL** format (`~/.config/vade/config.ccl`):

```ccl
view:
  default_mode: "long"       # "long", "short", "tree", "preview"
  show_hidden: false
  time_format: "%Y-%m-%d %H:%M"

theme:
  background: "#1e1e2e"
  foreground: "#cdd6f4"
  cursor: "#89b4fa"
  tag_mark: "#f38ba8"
  fast_key: "#f9e2af"
  directory: "#89b4fa"
  executable: "#a6e3a1"
  symlink: "#94e2d5"
  archive: "#fab387"

openers:
  ".dva": "nvim"
  ".c": "nvim"
  ".h": "nvim"
  ".md": "nvim"
  ".png": "xdg-open"
  ".jpg": "xdg-open"
  ".pdf": "zathura"
  default: "$EDITOR"
```

If no config file is present, `vade` boots with sensible, beautiful built-in defaults.

---

## 8. Implementation Milestones

1. **Phase 1 — Terminal Driver & Cell Grid Matrix**:
   - `term.dva`: Raw mode, alternate screen buffer, ANSI escape sequences, `TIOCGWINSZ`, double-buffered cell diffing renderer.
2. **Phase 2 — File System Model & Fast-Access Navigation**:
   - `fs.dva`: Directory reading, file statting, sorting, entry formatting.
   - `nav.dva`: Fast-access `a`–`z` letter indexing, cursor movement, double-tap detection.
3. **Phase 3 — Four View Modes & Substring Filter**:
   - Long list view, short list view, tree view, and live inline `/` substring filter.
4. **Phase 4 — Preview Engine**:
   - Built-in text pager with syntax highlighting, hex dump viewer for binaries, directory stats summary.
5. **Phase 5 — Selection & Async Task Runner**:
   - Multi-tagging (`Space`, `V`, `Ctrl-A`), async copy/cut/paste/delete worker queue with progress bar in bottom status line.
6. **Phase 6 — CCL Config, Openers & Polishing**:
   - `~/.config/vade/config.ccl` parsing, openers, help popup (`?`), edge case handling, and integration tests.
