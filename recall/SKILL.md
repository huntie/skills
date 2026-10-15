---
name: recall
description: Rebuild where a topic stands from the vault, the checkouts, and GitHub, and hand back a short current-state brief before work starts or resumes. Use when the user invokes /recall or says "catch me up", "where did I leave off", or "what's the state of X".
license: MIT
metadata:
  adapted-from: https://github.com/cursor/plugins/tree/main/pstack/skills/recall
disable-model-invocation: true
---

# Recall

Read-only. The deliverable is a brief, not a change, and the reading stays on the named topic and window. Heavy reads go to subagents where available; the main thread keeps findings and the brief.

1. Lock the scope before reading: the topic, the window ("recent" means the last seven days unless the user says otherwise), and which checkouts under `projects/` it touches. State the scope back in one line. If the user already gave a full state capsule, use it and skip the mining.
2. Read the vault for the topic: `vault/index.md`; every plan whose overview mentions it, with its status tag and `Branch:` line; the overlay under `vault/memory/<repo>/` for each checkout in scope; and the decisions log where the topic is devroot itself. A status tag is a claim to verify, not the answer.
3. Read live state in each checkout in scope: the current branch, the branches the plans name plus any whose names match the topic, commits in the window on them, WIP parking commits and stashes, and the working tree. In a Sapling checkout (`devroot.vcs=sapling`), also the draft stacks, `sl log -r 'draft()'` from inside the checkout, since a detached stack is on no branch. Then `gh pr list --author @me` and `gh pr view` for any PR those branches or plans name, with review and check state.
4. Reconcile. A plan or note is history; git and GitHub are current truth. Where they disagree, the brief says so and names the stale one. Chat history is not a source unless the user asks for it, and then only this workspace's own sessions.
5. Write the brief to the contract below, in plain prose, citing a plan path, a branch, or a PR number for each claim. Adjacent topics stay out unless they block this one. When it outgrows a screen, cut detail before cutting threads.

Contract:

- **Capsule.** At most five bullets: what the work is and where it stands overall.
- **Threads.** One line each, opening with exactly one tag: `[merged #N]`, `[open PR #N]`, `[in flight <branch>]`, `[verified, uncommitted]`, `[planned, not started]`, or `[shelved]`. Every thread gets a tag; an untagged thread is not done being investigated.
- **Problems.** At most five, the recurring ones, including anything tried and reverted so the next attempt starts where the last one failed.
- **Next move.** The single most useful concrete action.

If a plan's status tag disagrees with live state, end with the one-line correction and offer to update it per `vault/COMMITTING_POLICY.md`; do not update it unasked. When the next move is resuming a plan, name its branch and the `scripts/checkout <owner/repo>@<branch>` that lands on it.
