# harness-kit

Skills, hooks, and scripts that work across agent harnesses such as Claude Code and Codex.

## What is here

| Folder                                         | Holds                                                                                |
| ---------------------------------------------- | ------------------------------------------------------------------------------------ |
| [hooks/](hooks/open-ci-after-push)             | `open-ci-after-push`, a PostToolUse hook that opens the CI run after an agent pushes |
| [landline/](landline/SKILL.md)                 | Register a session with Landline, and enable read-only exchanges with matching peers |
| [neovide-operator/](neovide-operator/SKILL.md) | Show files in the Neovide window that matches the working directory                  |
| [operator/](operator/SKILL.md)                 | Turn a session into an operator that dispatches work to other agents                 |
| [tmux-operator/](tmux-operator/SKILL.md)       | Show files in an nvim session running in tmux                                        |
| [ux-designer/](ux-designer/SKILL.md)           | Evaluate and shape product interfaces                                                |

## Install a skill

Link the skill folder into each harness:

```sh
ln -s "$PWD/neovide-operator" ~/.claude/skills/neovide-operator
ln -s "$PWD/neovide-operator" ~/.codex/skills/neovide-operator
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

Licensed under the GNU General Public License, version 3 or later. See [LICENSE](LICENSE).
