# Sprint 3 Requirements

## Goal

Improve the visual experience of the TUI with syntax highlighting for command input, an enhanced sidebar, and a coherent cold-tone color palette inspired by Warp Terminal.

**Target platform:** macOS only

**Success criteria:** The terminal input displays syntax-highlighted commands with distinct colors for commands, flags, paths, strings, and numbers. The sidebar shows relative timestamps and optional Nerd Font icons. All UI components use a consistent, visually harmonious color palette.

---

## Core Features

### 1. Syntax Highlighting for Command Input

#### Token Types and Colors

| Token Type | Example | Color |
|------------|---------|-------|
| Command | `git`, `cargo`, `ls`, `cd` | Soft blue |
| Flag | `--verbose`, `-la` | Muted orange |
| Path | `/home/user`, `./file.txt` | Teal/cyan |
| String | `"commit message"` | Muted yellow |
| Number | `123`, `3.14` | Soft green |
| Operator | `\|`, `>`, `&&` | Accent cyan |
| Text | Other words | Default white |

#### Known Commands

The following commands are recognized and highlighted:

**Unix basics:** `ls`, `cd`, `pwd`, `mkdir`, `rmdir`, `rm`, `cp`, `mv`, `cat`, `less`, `more`, `head`, `tail`, `grep`, `find`, `chmod`, `chown`, `ln`, `touch`, `echo`, `clear`

**Git:** `git`

**Rust:** `cargo`, `rustc`, `rustup`

**Node/JS:** `npm`, `npx`, `node`, `yarn`, `pnpm`, `bun`

**Python:** `python`, `python3`, `pip`, `pip3`, `pipenv`, `poetry`

**System:** `sudo`, `su`, `man`, `which`, `where`, `env`, `export`, `source`

**Jerm:** `jerm`, `exit`, `quit`

#### Tokenizer Behavior

- First word in a command is always treated as a command
- Words starting with `-` are flags
- Words starting with `/`, `./`, `~/`, `..` or containing `/` are paths
- Words with file extensions (e.g., `.rs`, `.txt`) are paths
- Quoted text (`"..."` or `'...'`) is a string
- Pure numeric values are numbers
- After operators (`|`, `&&`, `;`), the next word is treated as a new command

---

### 2. Improved Sidebar

#### Layout

Current format:
```
1 ~/projects
2 ~/dev/rust
```

New format with relative time:
```
1  ~/projects         2h
2  ~/dev/rust         3d
```

#### Relative Time Display

| Duration | Display |
|----------|---------|
| < 60 seconds | `now` |
| < 60 minutes | `Nm` (e.g., `5m`) |
| < 24 hours | `Nh` (e.g., `2h`) |
| < 7 days | `Nd` (e.g., `3d`) |
| < 4 weeks | `Nw` (e.g., `2w`) |
| >= 4 weeks | `Nmo` (e.g., `1mo`) |

#### Nerd Font Icons (Optional)

Icons are disabled by default. Users can enable them via environment variable:

```bash
export JERM_NERD_FONTS=1
```

| Context | Nerd Font | Fallback |
|---------|-----------|----------|
| Folder | `` (nf-fa-folder_open) | `[D]` |
| Home folder | `` (nf-fa-home) | `[~]` |
| Git repo | `` (nf-dev-git) | `[G]` |

#### Border Style

Use straight-line ASCII box-drawing borders (`BorderType::Plain` in ratatui):
```
+----------+
|  content |
+----------+
```

---

### 3. Coherent Color Palette

A centralized cold-tone palette inspired by Warp Terminal.

#### Primary Colors

| Semantic | RGB | Usage |
|----------|-----|-------|
| `FG_PRIMARY` | `rgb(220, 230, 240)` | Default text |
| `BG_PRIMARY` | Terminal default | Background |

#### Accent Colors

| Semantic | RGB | Usage |
|----------|-----|-------|
| `ACCENT_PRIMARY` | `rgb(86, 156, 214)` | Primary highlights, focused borders |
| `ACCENT_SECONDARY` | `rgb(78, 201, 176)` | Teal/cyan accents, paths |
| `ACCENT_TERTIARY` | `rgb(156, 220, 254)` | Light blue highlights |

#### Syntax Colors

| Semantic | RGB | Usage |
|----------|-----|-------|
| `SYNTAX_COMMAND` | `rgb(86, 156, 214)` | Commands (blue) |
| `SYNTAX_FLAG` | `rgb(206, 145, 120)` | Flags (muted orange) |
| `SYNTAX_PATH` | `rgb(78, 201, 176)` | Paths (teal) |
| `SYNTAX_STRING` | `rgb(206, 206, 135)` | Strings (muted yellow) |
| `SYNTAX_NUMBER` | `rgb(181, 206, 168)` | Numbers (soft green) |

#### UI Colors

| Semantic | RGB | Usage |
|----------|-----|-------|
| `BORDER_DEFAULT` | `rgb(60, 70, 80)` | Normal borders |
| `BORDER_FOCUSED` | `rgb(86, 156, 214)` | Active/focused borders |
| `TEXT_MUTED` | `rgb(128, 140, 150)` | Subdued text, timestamps |
| `TEXT_SECONDARY` | `rgb(180, 190, 200)` | Secondary text |
| `SELECTION_BG` | `rgb(38, 50, 66)` | Selection background |
| `SELECTION_FG` | `rgb(220, 230, 240)` | Selection foreground |

#### Sidebar Colors

| Semantic | RGB | Usage |
|----------|-----|-------|
| `SHORTCUT_NUMBER` | `rgb(206, 206, 135)` | Shortcut numbers (1-9) |
| `TIME_AGO` | `rgb(128, 140, 150)` | Relative timestamps |

---

## Architecture

### New Modules

```
src/
├── theme/
│   ├── mod.rs           # Module exports
│   ├── colors.rs        # Palette struct with color constants
│   └── icons.rs         # IconPair struct, detect_nerd_font_support()
├── highlight/
│   ├── mod.rs           # Module exports
│   └── tokenizer.rs     # Token, TokenType, Tokenizer
```

### Modified Files

| File | Changes |
|------|---------|
| `src/main.rs` | Add `mod theme;` and `mod highlight;` |
| `src/ui/terminal.rs` | Use `Tokenizer` for input highlighting |
| `src/ui/sidebar.rs` | Add time_ago, icons, use palette |
| `src/ui/navigator.rs` | Migrate to palette colors |
| `src/app.rs` | Update `prompt_spans()` to use palette |
| `src/shortcuts/storage.rs` | Add `time_ago()` method to `Shortcut` |

---

## Out of Scope

The following are explicitly **NOT included** in Sprint 3:

- Theme configuration system (light/dark/custom themes)
- Status bar
- Visual autocompletion / suggestions
- Animations or visual feedback effects
- Frequency of use indicator in sidebar
- Git status per path in sidebar
- Directory size/file count in sidebar
- Cross-platform compatibility (Linux, Windows)

---

## Acceptance Criteria

### Syntax Highlighting

- [ ] Commands are highlighted in blue
- [ ] Flags (`-v`, `--verbose`) are highlighted in muted orange
- [ ] Paths (`/path`, `./file`, `~/dir`) are highlighted in teal
- [ ] Strings (`"text"`, `'text'`) are highlighted in muted yellow
- [ ] Numbers are highlighted in soft green
- [ ] Operators (`|`, `&&`, `>`) are highlighted in accent color
- [ ] First word after operators is treated as a new command
- [ ] Highlighting works with text wrapping

### Improved Sidebar

- [ ] Relative time is displayed right-aligned for each shortcut
- [ ] Time format follows the specified patterns (m, h, d, w, mo)
- [ ] Icons appear when `JERM_NERD_FONTS=1` is set
- [ ] Fallback ASCII/Unicode icons used when Nerd Fonts disabled
- [ ] Borders are straight lines (not rounded)
- [ ] Layout adapts when sidebar is narrow (hides time if < 20 chars)

### Color Palette

- [ ] All UI components use colors from the centralized palette
- [ ] No hardcoded colors remain in render functions
- [ ] Colors render correctly in Terminal.app and iTerm2
- [ ] Visual consistency across Terminal, Sidebar, and Navigator

---

## Suggested Implementation Order

1. **Theme module:** Create `src/theme/` with `colors.rs` and `icons.rs`
2. **Palette migration:** Update all UI files to use `Palette::*` constants
3. **Sidebar time_ago:** Add `time_ago()` method to `Shortcut`
4. **Sidebar icons:** Implement icon system with env var detection
5. **Sidebar layout:** Update render to show time and icons
6. **Highlight module:** Create `src/highlight/` with tokenizer
7. **Terminal integration:** Use tokenizer in `render_terminal()`
8. **Testing:** Unit tests and manual verification

---

## Testing

### Unit Tests

- `src/highlight/tokenizer.rs`: Test tokenization of various commands
- `src/shortcuts/storage.rs`: Test `time_ago()` edge cases
- `src/theme/icons.rs`: Test env var detection

### Manual Testing

```bash
# Test syntax highlighting
cargo run
$ git commit -m "fix: bug" --amend
$ cargo build --release 2>&1 | grep error
$ cd ~/projects/rust && ls -la

# Test Nerd Fonts
JERM_NERD_FONTS=1 cargo run

# Test in different terminals
# - Terminal.app
# - iTerm2
```
