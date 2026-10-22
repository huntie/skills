---
name: autopilot
description: Turn on autopilot for the rest of the session. The strategy is to continuously commit, when settled. Each unit of work is committed the moment it is proven to work, without asking, so a task builds a stack of atomic commits on its branch as it proceeds. Takes a rung (commit, push, submit) and `squash`, which tidies each task's stack at the end of the task. Use only when the user invokes /autopilot; `/autopilot off` turns it off.
license: MIT
disable-model-invocation: true
---

# Autopilot

**Continuously commit, when settled.** A commit lands at every unit boundary, once that unit is proven to work, so the branch is a stack of atomic commits by the end of the task: each holds one concern, stands on its own, and the stack reads as an argument for the change: every commit ends in a state a reviewer can check, in an order that makes the case.

Takes a rung, `squash`, or `off`, and each invocation restates the mode in full: `/autopilot push squash` sets both, `/autopilot commit` afterward drops squash. Invoking it is the explicit ask the standing rule against unasked commits requires, and the mode stays on until `off` or the end of the session. Say "Autopilot is on" in one line when it starts, naming the rung and squash when set; a conversation summary carries that line so the mode survives it.

## Rungs

Each rung includes the ones below it. The argument names how far the agent carries work without asking; everything above the current rung remains an explicit ask.

| Rung | Finished without asking | Reversibility |
|---|---|---|
| `commit` (default) | units on the task branch | local, fully reversible |
| `push` | the branch and its stack layers on `origin` | visible, reversible |
| `submit` | pull requests, draft first | reviewers are notified |

Only `commit` is defined below; `push` and `submit` get sections when a task first needs them.

## Settled

A unit is settled when its check has run and passed: the tests for what changed, and lint and typecheck where they apply. Green is the bar. A deliberately red commit, such as the failing test ahead of its fix, is allowed only when that red-then-green order is the delivery story.

## Continuously

Commit the moment a unit settles, and before the next one starts. The boundaries:

- A concern is complete and checked.
- The next edit belongs to a different concern. Finish and commit this one first.
- A prep step stands on its own: a rename, a deletion, a scaffold. It goes in its own commit ahead of the change that needs it.

Stage by path, never the whole tree. A commit holds one unit's files; changes already in the working tree when the mode started, and anything unrelated the user parked there, stay out.

## How to commit

- Before the first commit in a checkout, confirm the branch. Work in a checkout lands on a task branch per `AGENTS.md`, never on the default branch; if the checkout is still on its default branch, start the branch first with `scripts/pull` and `scripts/checkout <owner/repo>@<name> --new`. This repo, `vault/`, and `dotfiles/` commit on `main`. A Sapling checkout of a shared repo takes no named branch: commit drafts with `sl` on top of the synced default branch.
- Write every message with the commit-msg skill; it also carries the git and Sapling commands.
- Amend only the latest commit, only for a correction to it (a missed file, a lint fix, a typo), and only while it is unpushed. Anything else is a new commit. Reordering and squashing the stack are the user's call afterward, unless `squash` is on.
- When a `gh stack` is checked out, commit on the layer that owns the concern per the gh-stack skill. The `commit` rung produces commits, not layers. In a Sapling checkout, per `AGENTS.md`, each commit is a layer: commit with `sl` on the task branch, or on the draft stack itself in a shared repo.

## Squash

A task that iterates leaves follow-ups in its stack: the spacing tuned twice, the label renamed, the lint fix. With `squash` on, the stack is tidied once the task is done, through the squash skill: fix-ups fold into the commits they fix, distinct changes stay distinct, and the result may be one commit or several. Off by default, since it rewrites history the user has not seen.

- **What:** The unpushed commits since the tip the task started from. Record that tip when the task begins. A follow-up to a commit from before it stays its own commit. A task that touched several checkouts tidies each.
- **When:** At the end of the task, before the closing recap, and ahead of anything the `push` rung pushes. Never at unit boundaries: units land as they settle, so a red check mid-task leaves a stack to inspect.
- **Gate:** The squash skill's check decides. Red leaves the stack as it is, and the recap says so. A fold that is not a clear call is not made.

## Reporting

Name each commit as it lands, hash and title on one line, and list them all in the closing recap. With `squash` on, the recap lists the stack as it stands instead, hash and title per commit with the commits each absorbed. On `off`, stop committing, confirm the mode is off, and say what is left uncommitted in each checkout.
