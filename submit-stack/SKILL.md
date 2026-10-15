---
name: submit-stack
description: Promote a stack of local commits to stacked GitHub pull requests with gh stack, or sl pr submit in a Sapling checkout, one PR per commit, moving each commit message body into its PR description and leaving each commit with its title alone. Takes "draft" to open the PRs as drafts. Use only when the user invokes /submit-stack or asks to submit the commits as a stack; for a single commit, use submit.
---

# Submit stack

The multi-commit counterpart of the submit skill: each commit, prepared with commit-msg and usually tidied by squash, becomes one layer and one PR. Read the submit skill first; its rules for the description, the saved message, and outward-facing actions apply to every PR here unchanged. For `gh stack` itself, follow the gh-stack skill, including its non-interactive flags.

Takes an optional `draft` argument. `gh stack submit` opens drafts by default, so without `draft` pass `--open`.

## Preconditions

- Git, or Sapling where git config has `devroot.vcs=sapling`, which follows its own section below.
- Stacked PRs are same-repository: `origin` has to be where the PRs open. In a fork layout, where `upstream` exists and the PRs belong upstream, stop and say so; offer submit for the bottom commit instead.
- Two or more commits over the base. One: use submit. When a stack is already checked out (`gh stack view --json` succeeds), every layer holds exactly one commit; a layer with several stops the run, with squash offered to fold it.
- Staged and working tree changes stay untouched: rewrite messages with `--only`, and stash nothing.

## Steps

1. Read every commit bottom to top, `git log --reverse --format='%H' <base>..HEAD`, and save each full message to `$(git rev-parse --git-dir)/submit/<sha>.md`. Check which layers are already submitted with `scripts/pr-info <checkout> --branch <layer>`.
2. Drop the bodies, keeping each title:
   - **No stack yet:** Rebuild the chain on a detached head, as squash does: `git branch -f submit-backup HEAD`, `git checkout --detach <base>`, then for each commit `git cherry-pick --no-commit <sha>` and `git commit --only -m "<title>"`. Confirm `git rev-parse HEAD^{tree}` matches the old tip's tree; on a mismatch, `git reset --hard submit-backup` and stop.
   - **Existing stack:** For each layer, bottom to top: `gh stack checkout <branch>`, `git commit --amend --only -m "<title>"`, `gh stack rebase --upstack`. Return with `gh stack top`.
3. **No stack yet:** Create one branch per commit, bottom to top, each a brief kebab-case name from its title, with the owner prefix when `origin` is shared (the devroot branch rules apply). Point the original branch at the top commit so it stays the logical branch, then `gh stack init <bottom> … <original branch>` and check out the top.
4. Push and open the PRs: `gh stack submit --auto --remote origin`, plus `--open` unless `draft`. For an existing stack this pushes the rewritten layers with lease.
5. Set each new PR's title and description, since `gh stack` generates its own: build the description from the layer's saved message as the submit skill does, then `gh pr edit <number> --title "<title>" --body-file <file>`. Read the numbers from `gh stack view --json`. A layer that already had a PR is submitted, so follow the submit skill's resubmit rules instead: its description on GitHub is the source of truth, changed only by a revised summary and only in its summary section, and its title only when the commit's differs.
6. Delete `submit-backup` once every PR shows the right title and body.

## Sapling

`sl pr submit --stack` stands in for `gh stack`: the checkout's sl config, set by `scripts/checkout`, opens one PR per commit, each based on the branch of the PR below, the same shape `gh stack` makes, and keeps each PR's title and description on later submits. There are no layer branches to create and no backup to keep, since Sapling keeps every rewritten commit (`sl undo`).

A stack stays with the tool that opened it. When `gh stack view --json` succeeds, `gh stack` owns this one: follow the steps above, since `sl pr submit` opens a second PR for every layer it did not open. The one `gh stack` command a Sapling stack takes is `gh stack link`, which only registers existing PRs as a GitHub stack and keeps no local tracking.

1. List the stack bottom to top with `sl log -r 'draft() & ::.' -T '{node} {github_pull_request_number}\n'`; a number means that commit is already submitted. Save each full message (`sl log -r <node> -T '{desc}'`) to `$(git rev-parse --git-dir)/submit/<node>.md`.
2. Drop the bodies top to bottom, `sl metaedit -r <node> -m "<title>"`, so each edit restacks only commits already done.
3. Push and open with `sl pr submit --stack`, plus `--draft` for `draft`.
4. Register the PRs as a GitHub stack so the PR UI shows it: `gh stack link <numbers bottom to top> --remote origin` from inside the checkout. It keeps every PR number, branch and body; on a resubmit that added a layer, pass the stack number it printed first and the new PR after it. Later `sl pr submit` runs print `Cannot change the base branch because the pull request is part of a stack` for each PR and still push; the bases are already right.
5. List the stack again for the numbers, then set each new PR's description from its saved message as step 5 above does, with `gh pr edit <number> --body-file <file>`. Sapling appends a stack list to each description; keep it. A layer that already had a PR follows the submit skill's resubmit rules, including `scripts/pr-resolve-threads <checkout> --rev <node>` for the threads its push addressed.

## Reporting

The stack, bottom to top: branch, PR number and URL, and title, each PR's state (draft or open), and anything left for the user, such as a Test Plan with nothing to put in it. Say where the saved messages live.
