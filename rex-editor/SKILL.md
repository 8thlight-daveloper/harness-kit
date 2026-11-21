---
name: rex-editor
description: Use the user's nvim, in a Rex pane beside this session's tmux agents, as their IDE. Find the editor pane by Rex session and working directory, open files and changes in it over RPC, and reload buffers after edits. Use when the user types /rex-editor, asks to show a file, diff, ADR, or change in their editor, or after you create or edit a file the user should review.
---

# Rex editor

The user works in one Rex session per project. The editor is nvim in one Rex pane. This agent runs in a tmux window in
the pane beside it. Find that nvim, and show files there so the user can review them. Keep making edits with the Edit
and Write tools, never by typing into nvim.

When you create or edit a file the user should review, such as an ADR, contract, or scenario, open it in nvim. Do this
without being asked.

## Find the instance

Rex runs nvim in a terminal pane. nvim's RPC socket belongs to its `nvim --embed` child, not to the process Rex reports.
The socket is `nvim.<pid>.0` under `$TMPDIR/nvim.$USER/<random>/`. The random directory name is not the socket. The
socket moves on every restart, so find it each time.

`workbench` sets `REX_SESSION` and `REX_EDITOR_BLOCK` in the tmux session, so they name this project's Rex session and
editor pane. tmux panes otherwise inherit the Rex IDs of whoever started tmux, which can be another session, and a tmux
window that outlived its Rex session holds IDs that no longer exist. Check the IDs, then fall back to the tmux session
name, which `workbench` makes equal to the Rex session label.

```sh
DIR=$(pwd -P)   # use the project root when the session started in a subfolder

SESSION=$REX_SESSION
rex ls --json | jq -e --arg s "$SESSION" '.sessions[] | select(.session_id == $s)' >/dev/null 2>&1 || {
  LABEL=${TMUX:+$(tmux display-message -p '#S')}
  SESSION=$(rex ls --json | jq -r --arg l "$LABEL" '[.sessions[] | select(.label == $l)][0].session_id // empty')
}

BLOCKS=$REX_EDITOR_BLOCK
rex block ls -s "$SESSION" --json | jq -e --arg b "$BLOCKS" '.blocks[] | select(.block_id == $b)' >/dev/null 2>&1 ||
  BLOCKS=$(rex block ls -s "$SESSION" --json | jq -r '.blocks[].block_id')

printf '%s\n' "$BLOCKS" | while read -r B; do
  PID=$(rex block call -s "$SESSION" -b "$B" process | jq -r 'select(.foreground.name == "nvim") | .foreground.pid')
  [ -n "$PID" ] || continue
  CHILD=$(ps -axo pid=,ppid=,command= | awk -v p="$PID" '$2 == p && /--embed/ { print $1 }')
  find "${TMPDIR%/}/nvim.$USER" -name "nvim.$CHILD.0" 2>/dev/null | while read -r S; do
    CWD=$(nvim --server "$S" --remote-expr 'getcwd()' 2>/dev/null </dev/null) || continue
    [ "$CWD" = "$DIR" ] && echo "$S"
  done
done
```

- One match: use it.
- Several matches: ask the user which one.
- No match: say so and carry on without the editor. Do not launch nvim or create Rex panes.

If `rex` is not found, the shell's `PATH` lacks Rex's directory. `REX_BIN_DIR` holds it. Say so rather than guessing a path.

Run every `nvim --server` command with `</dev/null`. When stdin is a terminal, nvim starts a full-screen client and
prints escape codes instead of the answer.

Check the connection before relying on it. "Connection refused" means a stale path, so re-run discovery.

```sh
nvim --server "$S" --remote-expr 'getcwd() . " | " . expand("%:p") . " | mode=" . mode()' </dev/null
```

## Windows

List windows and buffer types before sending keys. A plain nvim has one editor window. A configured one may add a file
tree.

```sh
nvim --server "$S" --remote-expr 'join(map(range(1,winnr("$")), {_,n -> n . ":" . getbufvar(winbufnr(n), "&buftype") . ":" . bufname(winbufnr(n))}), " | ")' </dev/null
```

Open files in the first window with an empty `buftype`. Never in a file tree or a terminal window.

## Show a file

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:2wincmd w<CR>:edit +LINE /abs/path<CR>' </dev/null
```

Replace `2` with the editor window number found above. Use an absolute path and the line the user should look at. Do this
when you point the user at code, and when you start editing a file.

A file you just created exists on disk before you open it, so write it first. A buffer with unsaved user changes blocks
`:edit`. If the command fails, use `:split` in the editor window instead of forcing it.

## Reload after edits

After every Edit or Write to a file, refresh open buffers:

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:checktime<CR>' </dev/null
```

## Read state

Use `--remote-expr` to read the current file, cursor, or mode. Do not send keys to read state.

## Rules

- The user may be mid-edit. Send `<C-\><C-n>` first so keys land in normal mode. Do not change the layout.
- Never quit nvim (`:q`, `:qa`, `ZZ`, `ZQ`). Quitting closes the editor pane.
- Never send keys to the pane with `rex send` or `rex send-key`. Use nvim's RPC socket only.
- Saving is fine. Work is in git, so a save is recoverable. Do not save a buffer the user has unsaved changes in without
  asking.
- Closing a buffer is a judgment call. Close one you opened and no longer need. Leave the user's buffers alone, and never
  close the last editor window.
- Match on working directory only. Never send keys to an nvim for a different project, even one in another Rex session.
- Do not create, move, or close Rex panes, tabs, or sessions unless the user asks.
- If the editor is not nvim, or no socket matches, say so and carry on without it.
