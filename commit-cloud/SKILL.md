---
name: commit-cloud
description: Back up and restore draft Sapling stacks across machines with scripts/commit-cloud. Use when asked to back up a stack or work, to restore or resume work from another machine, or whether a stack is backed up, and before leaving a Sapling checkout for another machine. Optional and Sapling only; off until the first backup clones .commit-cloud/.
license: MIT
---

Draft Sapling stacks backed up from one machine appear on the others after their next `scripts/pull`, with hashes, dates and branches intact, beside that machine's own stacks and not checked out. Backing up is explicit; restoring happens on pull. Inspired by Meta's commit cloud, which open-source Sapling disables for git repos.

## Commands

- **Back up**: `scripts/commit-cloud backup [<checkout>...]`. Backs up every Sapling checkout under `projects/`, or those named, and restores what others backed up. Run it when asked to back up a stack, and before a machine is left. There is no per-stack mode: a checkout's draft heads are backed up as a set, which is what lets the set converge.
- **Restore only**: `scripts/commit-cloud restore <checkout>`, which `scripts/pull` runs for a Sapling checkout before returning to the branch or commit it started on. It also prints a `pending` line when the checkout's own stacks are not backed up, with the command to run.
- **Opt in**: the first explicit backup clones the repo to `.commit-cloud/`. Before that, `restore` and the pull hook do nothing. The repo must exist on GitHub, private and never a fork; `git config devroot.commitCloud` at the root names another as `owner/repo` or a URL.

## Reading the output

Each line is an action on one head, by its 12-character hash: `back up`, `retract`, `revive`, `restore … from <host>/<slot> as <branches>`, `bookmark … as <host>/backup-<date>`, `hide`, `keep`, `pending`. Exit 3 means something needs a human, and the message says what: the repo is unreachable, the push was rejected, or a stack's base is on no remote here.

## Resuming a restored stack

- It shows in `sl smartlog` as ordinary drafts, with the producer's task branch recreated and a bookmark `<host>/backup-<yyyymmdd>`, dated by the stack's tip commit, with `-1`, `-2` for more from the same host and day.
- Resume with `git switch <branch>` to commit on the task branch, or `sl goto <bookmark>` or `sl goto <hash>`. Nothing else changes the working copy.
- A restored stack is never backed up again by the machine that restored it. Work committed on top of it is new work, and the next backup carries it.

## How the set converges

- Hiding, amending or building on a stack anywhere retracts the old head on that machine's next backup: its bundle stays in the repo with a `<head>.hidden` tombstone beside it, and nothing is ever deleted there. The next restore on the others hides it, unless the working copy or one of the user's own branches sits on it there, in which case it says `keep` and names the reason; moving off, or deleting that branch, lets the next restore hide it.
- A stack hidden locally is never restored again from the repo, since the checkout remembers it. Unhiding it and backing up removes the tombstone, which revives it everywhere.
- A checked-out branch is never moved. When a restored head would move it, the branch stays and the stack arrives beside it.
- `sl hide` on a commit a git branch points at deletes that branch and hides the whole stack, with no parent left visible. Hiding the top of a task-branch stack therefore drops the stack, and does so even when a `git worktree` has the branch checked out.

## Never

- Edit or delete files in `.commit-cloud/` by hand, or commit there; the script owns it.
- Back up a git checkout with it: the script exits 1 for a named checkout without `devroot.vcs=sapling`. Pushed branches are already on the fork, and `scripts/checkout <owner/repo>@<branch>` fetches them elsewhere.
- Add a LaunchAgent or Sapling hook around it. Backing up stays an explicit action.
