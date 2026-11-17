---
name: neovide-operator
description: Use the user's Neovide, matched to the working directory, as their IDE. Find the instance over RPC, open files and changes in it, and reload buffers after edits. Use when the user types /neovide-operator, asks to show a file, diff, ADR, or change in Neovide, or after you create or edit a file the user should review.
---

# Neovide operator

The user keeps one Neovide window per project. Find the one whose working directory matches this session, and show files
there so the user can review them. Keep making edits with the Edit and Write tools, never by typing into Neovide.

When you create or edit a file the user should review, such as an ADR, contract, or scenario, open it in Neovide. Do
this without being asked.

## Find the instance

Neovide runs an embedded nvim per window. Its RPC socket is `nvim.<pid>.0` under `$TMPDIR/nvim.$USER/<random>/`. The
random directory name is not the socket. The socket moves on every restart, so find it each time.

A socket belongs to Neovide when the nvim's grandparent process is `neovide`. nvim in tmux has a different ancestry, so
this keeps `tmux-operator` instances out.

```sh
DIR=$(pwd -P)
for S in "$TMPDIR"nvim.$USER/*/nvim.*.0; do
  PID=${S##*/nvim.}; PID=${PID%.0}
  PP=$(ps -o ppid= -p "$PID" 2>/dev/null | tr -d ' ') || continue
  GP=$(ps -o ppid= -p "$PP" 2>/dev/null | tr -d ' ')
  ps -o command= -p "$GP" 2>/dev/null | grep -qi neovide || continue
  CWD=$(nvim --server "$S" --remote-expr 'getcwd()' 2>/dev/null) || continue
  [ "$CWD" = "$DIR" ] && echo "$S"
done
```

Use the project root as `DIR`, not a subfolder, when the session started in a subfolder.

- One match: use it.
- Several matches: ask the user which one.
- No match: say so and carry on without Neovide. Do not launch Neovide.

Check the connection before relying on it. "Connection refused" means a stale path, so re-run discovery.

```sh
nvim --server "$S" --remote-expr 'getcwd() . " | " . expand("%:p") . " | mode=" . mode()'
```

## Windows

List windows and buffer types before sending keys. The usual layout is window 1 for the neo-tree file tree and window 2
for the editor.

```sh
nvim --server "$S" --remote-expr 'join(map(range(1,winnr("$")), {_,n -> n . ":" . getbufvar(winbufnr(n), "&buftype") . ":" . bufname(winbufnr(n))}), " | ")'
```

Open files in the first window with an empty `buftype`. Never in the file tree or a terminal window.

## Show a file

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:2wincmd w<CR>:edit +LINE /abs/path<CR>'
```

Replace `2` with the editor window number found above. Use an absolute path and the line the user should look at. Do this
when you point the user at code, and when you start editing a file.

A file you just created exists on disk before you open it, so write it first. A buffer with unsaved user changes blocks
`:edit`. If the command fails, use `:split` in the editor window instead of forcing it.

## Reload after edits

After every Edit or Write to a file, refresh open buffers:

```sh
nvim --server "$S" --remote-send '<C-\><C-n>:checktime<CR>'
```

## Read state

Use `--remote-expr` to read the current file, cursor, or mode. Do not send keys to read state.

## Rules

- The user may be mid-edit. Send `<C-\><C-n>` first so keys land in normal mode. Do not change the layout.
- Never quit nvim or Neovide (`:q`, `:qa`, `ZZ`, `ZQ`). Quitting closes the user's window.
- Saving is fine. Work is in git, so a save is recoverable. Do not save a buffer the user has unsaved changes in without
  asking.
- Closing a buffer is a judgment call. Close one you opened and no longer need. Leave the user's buffers alone, and never
  close the last editor window.
- Match on working directory only. Never send keys to a Neovide for a different project.
- If Neovide is not running or no socket matches, say so and carry on without it.
