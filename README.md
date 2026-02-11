# git-wo

A thin wrapper around `git worktree` that creates worktrees in a ghq-style directory structure.

## Overview

`git-wo` organizes worktrees under `~/worktree/<host>/<user>/<repo>/<branch>`, making it easy to manage multiple worktrees across different repositories.

```
~/worktree/github.com/kfly8/my-project/feature1
~/worktree/github.com/kfly8/my-project/feature2
~/worktree/gitlab.com/team/other-repo/bugfix
```

## Installation

```bash
curl -o ~/bin/git-wo https://raw.githubusercontent.com/kfly8/git-wo/main/git-wo
chmod +x ~/bin/git-wo
```

Make sure `~/bin` is in your `PATH`.

## Usage

```bash
# Create a new worktree (creates new branch from current HEAD)
git wo add feature1

# Create a new worktree from a specific branch
git wo add feature1 main

# Pass options to git worktree add
git wo add feature1 main --track

# List worktrees
git wo list

# Remove a worktree
git wo remove feature1

# Remove all worktrees for merged branches
git wo prune-merged

# Prune stale worktree info
git wo prune
```

## Sandbox

`git-wo` can automatically launch a sandboxed sub-shell when you `cd` into a worktree directory. The sandbox uses macOS `sandbox-exec` (Apple Seatbelt) to restrict file writes to the worktree directory only, keeping the rest of your home directory safe.

### Setup

Add to your `~/.zshrc` (or `~/.bashrc`):

```bash
eval "$(git-wo init)"
```

### How it works

When you `cd` into a worktree under `~/worktree/...`, a sandboxed sub-shell is automatically started:

```bash
cd ~/worktree/github.com/user/repo/feature1
# → "Entering sandbox for: /Users/user/worktree/..."
# All commands now run inside the sandbox

touch ~/should-fail  # → Operation not permitted
touch ./local-file   # → OK (worktree directory is writable)

exit
# → "Left sandbox" (back to original shell)
```

### Disabling sandbox

To `cd` into a worktree without sandbox:

```bash
GIT_WO_NO_SANDBOX=1 cd ~/worktree/.../feature1
```

### Notes

- **macOS only**: `sandbox-exec` is required. On other platforms, the shell integration is a no-op.
- **Nesting prevention**: The sandbox will not start if you are already inside one (`$GIT_WO_SANDBOX`).
- **Custom profile**: Set `wo.sandboxProfile` in git config to use a custom sandbox profile.

## Configuration

Configure via `git config`:

```bash
# Set worktree root directory (default: ~/worktree)
git config --global wo.root ~/worktree

# Set post-create hook script
git config --global wo.postHook ~/.config/git-wo/post-hook

# Set custom sandbox profile (default: <script-dir>/sandbox/git-wo.sb)
git config --global wo.sandboxProfile /path/to/custom.sb
```

### Post Hook

The `wo.postHook` script runs after creating a worktree. Environment variables available:

- `ORIGINAL_PATH`: Path to the original repository
- `WORKTREE_PATH`: Path to the new worktree
- `BRANCH_NAME`: Name of the new branch

Example hook script (`~/.config/git-wo/post-hook`):

```bash
#!/bin/bash
cp "$ORIGINAL_PATH/.envrc" "$WORKTREE_PATH/" 2>/dev/null
direnv allow "$WORKTREE_PATH"
mise trust "$WORKTREE_PATH"
```

Don't forget to make it executable: `chmod +x ~/.config/git-wo/post-hook`

## Tips

### fzf integration

Add to your `.zshrc` to jump between worktrees with `Ctrl-G`:

```bash
function fzf-worktree () {
  local selected=$(git worktree list 2>/dev/null | fzf --query="$LBUFFER" | awk '{print $1}')
  if [ -n "$selected" ]; then
    BUFFER="cd ${selected}"
    zle accept-line
  fi
  zle reset-prompt
}
zle -N fzf-worktree
bindkey '^g' fzf-worktree
```

## License

MIT
