---
name: tmux-operator
description: Use the user's nvim, running in tmux, as their IDE. Open files and changes in nvim over RPC, and reload buffers after edits. Use when the user types /tmux-operator, or asks to show a file, diff, or change in their editor.
---

# Tmux operator

The user's nvim is their IDE. This session runs in a tmux pane beside it. Show files and changes there so the user can
navigate and review them. Keep making edits with the Edit and Write tools, never by typing into nvim.

## Find nvim

Use the `default` tmux socket. The nvim session is named `operator-<slug>`.

```sh
tmux -L default list-sessions
tmux -L default capture-pane -p -t <session>:workspace
```

nvim's RPC socket moves on every nvim restart. Find it each time:

```sh
S=$(ls "$TMPDIR"nvim.$USER/*/nvim.*.0 | head -1)
```

The directory name (for example `VxKvsg`) is not the socket. The socket is the `nvim.<pid>.0` file inside it. If
several match, pick the one whose pid is the `nvim --embed` child of the session's nvim.

Check the connection before relying on it:

```sh
nvim --server "$S" --remote-expr 'expand("%:p") . " win=" . winnr()'
```

If it fails with "connection refused", the path is stale. Re-run the discovery.

## Windows

List windows and buffer types before sending keys. The usual layout is window 1 for the neo-tree file tree and window
2 for the editor.

```sh
nvim --server "$S" --remote-expr 'join(map(range(1,winnr("$")), {_,n -> n . ":" . getbufvar(winbufnr(n), "&buftype") . ":" . bufname(winbufnr(n))}), " | ")'
```

Open files in the editor window. Never in the file tree or a terminal window.

## Show a file

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:2wincmd w<CR>:edit +LINE /abs/path<CR>'
```

Use an absolute path and the line the user should look at. Do this when you point the user at code, and when you start
editing a file.

## Reload after edits

After every Edit or Write to a file, refresh open buffers:

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:checktime<CR>'
```

## Read state

Use `--remote-expr` to read the current file, cursor, or mode. Do not send keys to read state.

## Rules

- The user may be mid-edit in nvim. Send `<C-\><C-n>` first so keys land in normal mode. Do not change the layout.
- Never quit nvim (`:q`, `:qa`, `ZZ`, `ZQ`). Quitting shuts down part of the IDE.
- Saving is fine. Work is in git, so a save is recoverable. Do not save a buffer the user has unsaved changes in
  without asking.
- Closing a buffer is a judgment call. Close one you opened and no longer need. Leave the user's buffers alone, and
  never close the last editor window.
- If nvim is not running or no socket is found, say so and carry on without it.
