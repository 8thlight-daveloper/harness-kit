# harness-kit

Skills, hooks, and scripts that work across agent harnesses such as Claude Code and Codex.

## What is here

| Folder                                         | Holds                                                                                |
| ---------------------------------------------- | ------------------------------------------------------------------------------------ |
| [bin/](bin/workbench)                            | `workbench`, which opens a Rex session with your editor and a tmux session of agents |
| [completions/](completions/_workbench)         | `_workbench`, zsh completion for `workbench` project names                           |
| [hooks/](hooks/open-ci-after-push)             | `open-ci-after-push`, a PostToolUse hook that opens the CI run after an agent pushes |
| [landline/](landline/SKILL.md)                 | Register a session with Landline, and enable read-only exchanges with matching peers |
| [neovide-operator/](neovide-operator/SKILL.md) | Show files in the Neovide window that matches the working directory                  |
| [operator/](operator/SKILL.md)                 | Turn a session into an operator that dispatches work to other agents                 |
| [rex-operator/](rex-operator/SKILL.md)         | Show files in the nvim pane of the Rex session that runs this agent                  |
| [tmux-operator/](tmux-operator/SKILL.md)       | Show files in an nvim session running in tmux                                        |
| [ux-designer/](ux-designer/SKILL.md)           | Evaluate and shape product interfaces                                                |

## Install a skill

Link the skill folder into each harness:

```sh
ln -s "$PWD/neovide-operator" ~/.claude/skills/neovide-operator
ln -s "$PWD/neovide-operator" ~/.codex/skills/neovide-operator
```

`rex-operator` goes with `workbench`, which gives the tmux session the `REX_SESSION` and `REX_EDITOR_BLOCK` IDs
that the skill reads to find the editor pane.

## Install the scripts

Link `workbench` into a directory on your `PATH`:

```sh
ln -s "$PWD/bin/workbench" ~/bin/workbench
```

### workbench

`workbench` opens a Rex session named after the project folder, with one Editor tab rooted at the folder. No other app
opens. Running it again for the same project reuses the session.

- **Left side:** `$EDITOR`, or nvim when it is unset. It may include arguments, such as `nvim -u NONE`.
- **Right side:** 35 percent of the width, attached to a tmux session of the same name with Claude, Codex, and CLI
  windows. Switch windows with tmux's own keys. Change `agents_percent` at the top of the script to resize it.

The tmux session outlives the Rex session. If you close the Rex session and run `workbench` again, the new Rex session
attaches to the same tmux windows and their running agents.

It needs `rex`, `jq`, `tmux`, `landline`, `claude`, `codex`, and your editor on your `PATH`. Selecting the new session
in the Rex app needs Remote Control. Without it, the script says so and you pick the session yourself.

Do not bind a key sequence that starts with your tmux prefix in Rex's `init.lua`. Rex consumes the first key of a
sequence, so tmux would never see the prefix.

Set `WORKBENCH_HOME` to the folder that holds your projects, and `workbench` opens one by name from anywhere:

```sh
export WORKBENCH_HOME=~/Projects
workbench harness-kit
```

A bare name is looked up under `WORKBENCH_HOME` first. A path that starts with `/`, `./`, or `../`, or is `.` or `..`,
always means that path. With no argument, `workbench` uses the current directory.

For zsh completion of project names, link `completions/_workbench` into a directory on your `fpath`, then run
`compinit`:

```sh
ln -s "$PWD/completions/_workbench" ~/.zsh/completions/_workbench
```

If a generated shell config, such as home-manager's, runs `compinit` before your own init, register the function
afterward instead. No second `compinit` is needed:

```sh
fpath+=(/path/to/harness-kit/completions)
autoload -Uz _workbench
compdef _workbench workbench
```

## Use the CI hook

`hooks/open-ci-after-push` reads a hook payload on standard input, or takes `--command` and `--cwd`. After a successful
`git push` to a GitHub remote, it waits for the CI run on the pushed commit and opens it in a browser. Any other
command, host, or directory exits silently. Run it with `--help` for the settings.

Claude Code, in `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "/path/to/harness-kit/hooks/open-ci-after-push" }] }
    ]
  }
}
```

Codex, in `~/.codex/config.toml`:

```toml
[[hooks.PostToolUse]]
matcher = "^Bash$"

[[hooks.PostToolUse.hooks]]
type = "command"
command = "/path/to/harness-kit/hooks/open-ci-after-push"
```

It needs `git`, `gh`, and Python 3. Set `OPEN_CI_OPENER` to choose the browser. The default is `open -a Zen` on macOS.

## License

Copyright © 2026 8th Light, LLC. Licensed under the GNU General Public License, version 3 or later. See
[LICENSE](LICENSE).
