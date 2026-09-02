---
name: rebase
description: Rebase the current branch or stack onto its upstream target, in git or Sapling/Mercurial. Use only when the user invokes /rebase or explicitly asks to rebase — it rewrites history, so never start it on inference.
---

Takes an optional argument: a branch or bookmark name. Fall back to the defaults below when none is given.

## Mercurial (sl/hg)

Default bookmark: `remote/fbsource/stable`.

1. `sl commit -m "WIP $(date '+%Y-%m-%d %H:%M')" || true`
2. `sl pull --rebase -d <bookmark>`
3. Resolve conflicts if any, then `sl rebase --continue`.

## Git

Default branch: `main`.

1. `git stash --include-untracked`
2. `git fetch --all`
3. Determine the rebase target: use `upstream/<branch>` if the `upstream` remote exists, otherwise `origin/<branch>`.
4. `git rebase <target>`
5. Resolve conflicts if any, then `git rebase --continue`.
6. `git stash pop || true`
