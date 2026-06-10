# read-it-fast (terminal edition)

RSVP speed reading for the terminal, inspired by the
[read-it-fast](https://github.com/codyadam/read-it-fast) browser extension.
Select any text in any tmux pane — Claude Code, nvim, `bat`, `man`, a log —
press one key, and read it word-by-word in a popup with the
optimal-recognition-point letter highlighted, Spritz-style.

```
            ────────────┬────────────

                     pun c tuation,      ← ORP letter in red, eyes never move

            ────────────┴────────────

 350 wpm · 14/26 (53%)
```

No dependencies: `rif` is a single-file Python 3 (stdlib only) script,
`rif-tmux` is POSIX sh.

## Install

```sh
install -m755 rif rif-tmux ~/.local/bin/
echo 'source-file ~/projects/read-it-fast/read-it-fast.tmux.conf' >> ~/.tmux.conf
tmux source-file ~/.tmux.conf
```

## Usage in tmux

| Action | How |
|---|---|
| Read a mouse selection | Drag-select, then press `R` while selection is active |
| Read the last copy | Drag-select (auto-copies on release), then `prefix + R` |
| Read the visible pane | `prefix + P` (great for Claude Code answers) |

Why no true "hover"? Terminals don't expose hover events over arbitrary app
content, so select-then-keypress is the closest ergonomic equivalent. tmux is
the integration layer, which is what makes this work identically inside any
program running in a pane.

## Usage anywhere

```sh
man tmux | rif           # pipe anything
rif notes.txt            # or read a file
rif -w 500 notes.txt     # custom speed (default 350, or $RIF_WPM)
```

### Neovim

Speed-read a visual selection (inside tmux it opens the popup):

```lua
vim.keymap.set('x', '<leader>R', ':w !rif-tmux<CR>', { desc = 'Speed-read selection' })
```

## Keys (inside the reader)

| Key | Action |
|---|---|
| `space` | pause / resume — pausing shows surrounding context, resuming re-ramps speed |
| `h` / `l`, `←` / `→` | previous / next word |
| `,` / `.` | previous / next sentence |
| `j` / `k`, `↓` / `↑` | slower / faster (±25 wpm) |
| `+` / `-` | bigger / smaller text |
| `g` | restart |
| `q` / `esc` | quit |

## Text size

Terminals can't change their font size from inside a program, so sizes 2-5
render words as big block glyphs (a built-in 5x7 bitmap font from Adafruit-GFX,
drawn with Unicode half-blocks), with the ORP letter still colored. Size 1 is
plain terminal text. Words too big for the window auto-shrink to fit.

Speed and size persist across runs in `~/.config/rif/config` (or
`$XDG_CONFIG_HOME/rif/config`) — whatever you set with the keys is what you
get next time. `-w`/`-s` flags override the saved values for one run.

## Behavior

- ORP highlighting per word length (1 char → 1st letter … 14+ → 5th)
- Longer pauses on sentence ends (×2.2), clauses (×1.6), paragraphs, numbers,
  and long words; words over 18 chars are hyphen-chunked
- Gentle speed ramp-up after every start/resume
- ANSI escape codes stripped from input, so piping colored output is fine

## Prior art

[pasky/speedread](https://github.com/pasky/speedread) pioneered terminal RSVP;
this adds ORP-pivot rendering, pause-with-context, sentence navigation, and the
tmux mouse-selection → popup workflow.
