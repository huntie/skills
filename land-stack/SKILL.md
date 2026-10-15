---
name: land-stack
description: Land a stack of approved GitHub pull requests by squash-merging them one at a time, bottom first, so each PR becomes its own commit on the base branch. Takes the top PR number or a GitHub stack number. Use only when the user invokes /land-stack.
license: MIT
disable-model-invocation: true
---

# Land stack

Merges a stack PR by PR, never with GitHub's "Squash and merge stack" button, which folds every layer into one operation. Each layer lands as its own squash commit, and the next layer is checked before it merges.

Merging is outward-facing and cannot be undone, so this runs only on the user's explicit request, for the stack they named.

## Before merging

1. List the stack bottom to top: `gh stack view --json` when the stack is checked out, else read each PR's `baseRefName` from `gh pr view <n> --json number,title,baseRefName,mergeable,mergeStateStatus,reviewDecision,isDraft` and follow the bases down to the trunk.
2. Stop and report when any layer is a draft, not `MERGEABLE`, or has a `reviewDecision` other than `APPROVED`. `UNSTABLE` alone is fine: it means a non-required check is pending or failing, and branch protection decides at merge time.
3. Tell the user the order and the PR numbers in one line, then start; do not ask again.

## Merging

For each layer, bottom first:

1. `gh stack merge <pr> --yes --squash` merges that PR alone, since it is the bottom of what remains. In a repository with no GitHub stack, `gh pr merge <pr> --squash` does the same. Use the merge method the user named, if any.
2. Confirm it: `gh pr view <pr> --json state,mergeCommit` reads `MERGED`, and note the merge commit.
3. Before the next layer, wait a few seconds for GitHub to retarget it, then confirm its base is the trunk and its `files` are only its own. A layer that still carries the merged layer's changes, or shows `CONFLICTING`, stops the run.

Stop at the first failure and report what merged and what did not. Never bypass branch protection, and never fall back to merging the remaining layers together.

## After

- Report each PR with its merge commit, bottom to top.
- Update the plan that names the branch: its `Status:` tag becomes `[merged #<bottom>]`, or the per-PR numbers when there are several.
- In the checkout, `scripts/pull` to bring the trunk level. The local stack's commits are now on the trunk as squash commits; leave the task branch for the user to delete.
