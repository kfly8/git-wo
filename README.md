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

## Environment Variables

- `WORKTREE_ROOT`: Base directory for worktrees (default: `~/worktree`)

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
