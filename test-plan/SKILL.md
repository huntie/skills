---
name: test-plan
description: Decide what to run to verify a change and write the test plan, what was run and what it proved, for a commit message or PR description. Use when writing a commit message or PR description, when asked for a test plan or how to verify a change, or once a verification finishes. Asks which checks the user wants, with options fitted to the change and a smoke test as the fallback, unless the plan, the request or the session already settles it.
license: MIT
---

# Test plan

A test plan shows what was run and what it proved, concrete over prose. In a commit message it is a `**Test plan**` block after the summary; the submit skill lifts it into the PR's Test Plan section. The commit-msg skill writes the summary; this skill writes this.

## Decide what to run

Ask only when the answer is not already there. It is already there when the task's plan under `vault/plans/` names the verification, the user asked for specific checks, the session has already run what the change needs, or the change is docs or copy with nothing to run. Otherwise ask once, before verifying, with two to four options fitted to the diff, each a one-line plan the user can pick, multi-select where they combine. Use AskUserQuestion where the harness has it, else a numbered list in chat. No answer, as in a non-interactive session, means the fallback.

Options to draw from, by what the diff touches:

- **Unit tests** for the touched package: the specific files, `yarn jest path/to/file.test.ts` or the repo's equivalent, never the whole suite.
- **Screenshots per state** on a simulator, through the simulator-screenshots skill where it is installed: one framed shot per state the change cares about, on iOS and Android as the change warrants, for anything visual.
- **Smoke test**, the fallback: build and launch a representative app on one device per affected platform, then one framed screenshot labelled by device with `✅ App builds and bundles`. For a change with nothing to see that can still break the build, bundling or startup: native code or targets, config plugins, dependencies and lockfiles, bundler setup. In a stack, each such layer gets its own, captured with that layer checked out.
- **A check only the user can drive**, written as the steps for them.
- **CI only**, said alone, when nothing is worth running locally.

A repo often has its own smoke-test app, device and setup steps. Look in the vault first: `vault/index.md` lists memory notes per repo, and one named for the smoke test or the app to build settles the choice. Where the vault has none and the choice was not obvious, the user picks the app.

## Write it

- Commands in fenced blocks; `✅`, `⚠️` or `ℹ️` to mark outcomes; a Before/After table when a comparison needs one.
- Leave out lint, format and type-check results; CI runs those. No pass-count lines such as "101 tests pass" under a test command either: the command shows what ran.
- **Screenshots**: one per state, labelled by device and state, in a table whose column headers are the labels, a pair side by side and a lone shot in a one-column table too. Each outcome sits directly under the shots that show it, not in one list ahead of every table.
- **Follow-up layer**: a layer that re-verifies the preceding PR opens with `Repeat all verification from <PR link>`, then carries only what is new to this layer, its own shots and checks. Setup snippets, test commands and caveats the preceding plan already states are not repeated. A shot that shows the change rather than proves a state belongs in the summary, per the commit-msg skill.
- **Images** are `<img src="…" width="…" />`, 300 for a portrait phone frame and 400 for a landscape or wide one, never a bare Markdown image, whatever a repo template says about full-width images. A commit message references the files, kept somewhere that outlives the session such as `scratch/<task>-evidence/`; the submit skill uploads them when the PR opens.
- State the bar honestly: if verification was mechanical rather than end-to-end, say so, and name what you did not run and why. When CI genuinely is the test, say that alone.
