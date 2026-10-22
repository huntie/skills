---
name: handoff
description: Write a handoff of the current session to vault/handoffs/ so it can be resumed on another machine, under a slug the skill picks. Use only when the user invokes /handoff.
license: MIT
disable-model-invocation: true
---

# Handoff

A handoff is for a fresh session on another machine with no chat history. That session gets the vault and the pushed state of each checkout, and nothing else. Write the file so a reader seeing it cold can continue the work without asking what came before.

Takes an optional argument: a slug, or a hint about what to stress.

1. Run `git -C vault pull --rebase --autostash`. If `vault/handoffs/README.md` is missing, copy this skill's `templates/README.md` there.
2. Pick the slug: two to five kebab-case words naming the work, not the session (`metro-stale-graph-fix`, not `debugging-session`). The file is `vault/handoffs/<YYYY-MM-DD>-<slug>.md`, dated today. If that path exists from an earlier handoff of the same work, update it in place.
3. Collect live state for each checkout the session touched: the branch, `git status --short`, and commits not on the remote (`git log @{u}..` or, with no upstream, the branch has never been pushed). In a Sapling checkout, also `sl log -r 'draft()'` from inside it. None of that travels. List it under **Not pushed**, then tell the user before writing. Do not commit or push it for them.
   - When commit cloud is set up (`.commit-cloud/.git` exists at the root) and a touched Sapling checkout has draft stacks, ask the user whether to back them up now with `scripts/commit-cloud backup <checkout>`, per the commit-cloud skill, so the other machine restores them on its next pull. Run it only on a yes. Note under **Not pushed** which stacks are backed up and which are not.
4. Write the file with these sections, leaving out any that would be empty:
   - **Goal.** One or two sentences on what the work is for, plus a link to the plan or the PR if there is one.
   - **State.** Where things stand: what is done and verified, and what is in progress. Cite branches, PR numbers, SHAs, and file paths.
   - **Checkouts.** One line per checkout: `projects/<name>`, its branch, and the `scripts/checkout <owner/repo>@<branch>` command that lands on it.
   - **Not pushed.** From step 3.
   - **Decisions.** What was chosen in this session and why, including the user's preferences and corrections that apply to this work.
   - **Dead ends.** What was tried and failed, and why, so the next session does not repeat it.
   - **Open questions.** Anything waiting on the user or still unknown.
   - **Next step.** The single most useful concrete action.
   - **Resume prompt.** One fenced block to paste into the new session. It points to this file and states the next step.
   Keep it to what the session learned. Anything the code, a plan, or `git log` already shows gets a link, not a repeat. Leave out secrets and tokens.
5. Do not add the file to `vault/index.md`. Commit it on its own as `handoffs: Add <slug>` (or `handoffs: Update <slug>`), with the README in the same commit the first time, then push per `vault/COMMITTING_POLICY.md`.

End with the file path, the resume prompt, and anything listed under **Not pushed**.
