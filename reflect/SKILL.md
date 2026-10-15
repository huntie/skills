---
name: reflect
description: File what a session taught into vault/, a skill, or a script, applying the vault's write bar. Use at the end of a task when the user corrected the session, a decision changed how a repo's setup or workflow works, or a gotcha cost real time and is not recorded in the code. Also on "reflect" or "remember this". Without one of those triggers, do not use it.
license: MIT
metadata:
  adapted-from: https://github.com/cursor/plugins/tree/main/pstack/skills/reflect
---

# Reflect

This is the write path into `vault/` for decisions and lessons. Filing nothing is the expected outcome of most runs, and the summary says so. One-offs are not learnings.

1. Read `vault/README.md` for the layout and the bar, then `vault/index.md`. Read any existing note or skill a candidate could extend before writing.
2. Scan the conversation for candidates: corrections received, decisions that changed a workflow, gotchas that cost real time, dead ends and the path that worked when it generalizes, friction in a skill or script this workspace owns, a skill that should have fired and did not, repeated manual steps a script could absorb.
3. Scrutinize each candidate, in order, and drop it at the first failure:
   - State it as one sentence naming the future task it changes. No such sentence, no note.
   - Durable: still true in six months once paths, SHAs, and tool versions have changed. A platitude fails too; so does a fact that drifts.
   - Drop anything the code, git history, or a few minutes of reading shows on its own. The README's worked example is the calibration.
   - Drop anything already captured. If an existing note or skill covers it partly, the result is an edit that makes it fire, wording or placement, not a new one.
4. Decide each survivor's route without writing yet, in this order: a script or check that enforces it; the description of a skill that existed but did not trigger; the skill in `skills/` it is about; the plan it belongs to; a new `skills/principle-<name>/SKILL.md` when the rule holds across repos and no skill can hold it, in the shape of the existing principle skills; or, for a note about one repo, the line that repo's own agent instructions would carry.
5. Present every proposal as one tick list and wait. Use the harness's multiple-choice question with multiple selection where it exists, in batches of four; otherwise a numbered list in prose. Each item is one line: the route and target, then the change or the drafted note, with the recommended items marked. A repo note carries its own destination choice beside it: upstream, as a PR to the repo's `AGENTS.md` or `CLAUDE.md` on a task branch or the strongest check the repo would accept; or the overlay under `memory/<repo>/`, for what the repo cannot hold. Dropped candidates are not options; they go in the summary with their reason.
6. Apply only what was ticked, then commit each vault write on its own. While touching a repo's overlay, graduate or delete any note that now fails the test: cited twice means it becomes a skill or a script, and one the repo would take goes upstream. A lesson stays under fifteen lines, and a note a survivor supersedes is deleted.

End with a short summary: what was applied and where, one line each; what was offered and declined; and every candidate dropped before the list, with the reason.
