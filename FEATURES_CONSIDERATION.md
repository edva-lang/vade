# High-Value Features Warranting Consideration for Vade

This document analyzes high-value capabilities from established terminal file managers (`nnn`, `ranger`, `yazi`, `mc`) and outlines how they align with Vade's design principles in Edva: zero-overhead, deterministic memory, responsive keyboard navigation, and robust systems-level abstractions.

---

## 1. Paradigm Overview

| File Manager | Primary Paradigm | Core Strength | Key Architectural Insight |
|---|---|---|---|
| **`nnn`** | Ultra-minimalist, C-native | Extreme speed (<1MB RAM), plugin architecture via shell IPC, 4 independent contexts | Zero overhead, terminal-first shell handoff |
| **`ranger`** | Miller columns, Python | Smooth vi-keys, tabbed navigation, extensible rifle file opener | Intuitive visual directory hierarchy |
| **`yazi`** | Modern asynchronous, Rust | Non-blocking async I/O, Lua plugins, terminal graphics (sixels/kitty/ueberzug) | Async indexing & instant responsiveness |
| **`mc`** | Dual-pane Norton Commander | Side-by-side transfers, VFS (tar/zip/ftp/ssh inspection), subshell integration | Precision file manipulation & archive browsing |

---

## 2. High-Value Features for Vade

### 1. In-Archive VFS Virtual Browsing (from `mc`)
- **Concept**: Transparently treating `.tar`, `.tar.gz`, `.zip`, and `.ccl` files as virtual directories. Pressing `Enter` on `archive.tar.gz` navigates into its directory tree, showing contents and previewing files inside without extracting to disk.
- **Why it fits Vade**: `std/ccl` already exists in Edva, and Edva's streaming `read_to_string` / buffer slicing allows memory-mapped or piped archive listing (`tar -tf`, `unzip -l`). Vade's `list_dir` abstraction can dispatch to virtual entry generators based on path prefixing (e.g. `/path/to/archive.tar.gz::/subfolder`).

### 2. Multi-Context Workspaces / Tabs (from `nnn` & `ranger`)
- **Concept**: 4 fixed numbered tabs/workspaces (switched with `Tab` or `Alt-1..4`). Each context maintains an independent working directory, cursor index, scroll offset, and filter string. Selection carts can either be global (cross-workspace) or local.
- **Why it fits Vade**: Staging batch copy/move operations is much faster when users can jump between source and destination directories without losing navigation state or cursor position.

### 3. Intelligent Opener / "Rifle" Rule Engine (from `ranger` & `yazi`)
- **Concept**: Declarative rules mapping file extensions, MIME types, and attributes to default viewer/editor commands, with graceful fallbacks (e.g. GUI image viewer if `$DISPLAY` or `$WAYLAND_DISPLAY` is present; CLI pager/viewer like `viu` or `less` otherwise; `$EDITOR` for text).
- **Why it fits Vade**: Currently Vade uses fixed editor invocations (`$EDITOR`) or direct execution. A simple opener configuration in CCL (e.g. `~/.config/vade/opener.ccl`) parsed via `std/ccl` provides clean extensibility without external scripting dependencies.

### 4. Asynchronous Background Directory Sizing (`du`) (from `yazi`)
- **Concept**: Calculating total directory disk consumption in the background without blocking the UI thread or terminal event loop.
- **Why it fits Vade**: Vade already utilizes non-blocking event polling. A lightweight background worker or non-blocking recursive traversal using `fs::read_dir` can update directory sizes incrementally in the status bar or list view.

### 5. Advanced Filtering & Glob/Fuzzy Matching (from `nnn` & `yazi`)
- **Concept**: Expanding `/` live filtering to support wildcard glob patterns (`*.dva`, `src/`) and negative exclusions (`!*.o`).
- **Why it fits Vade**: Vade already has an interactive live search prompt (`PROMPT_FILTER`). Upgrading the matching function to handle simple globs and inverted filters makes navigating large codebases instant.

### 6. Bulk / Batch Renaming via External Editor (from `ranger` & `nnn`)
- **Concept**: Staging selected files into a temporary text buffer, opening `$EDITOR`, and applying edits line-by-line as bulk `fs::rename_path` operations upon save.
- **Why it fits Vade**: Reuses `std/fs::write_file`, `std/fs::read_to_string`, and `std/fs::rename_path`. Provides a powerful, friction-free way to rename dozens of files using familiar editor macros or search-and-replace.

### 7. Subshell Environment Handoff (from `mc`)
- **Concept**: Pressing `Ctrl-O` or backtick suspends the TUI and drops into an interactive shell in the currently viewed directory, while retaining Vade's state. Exiting the shell instantly resumes Vade in that directory.
- **Why it fits Vade**: Complements the existing `-c / --cwd-file` flag by allowing immediate in-session terminal execution without quitting Vade.
