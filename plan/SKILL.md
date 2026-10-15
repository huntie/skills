---
name: plan
description: Write a numbered plan under vault/plans/ for work the user wants planned before it is built. Planning only, never implementation. Use only when the user invokes /plan or asks for a plan.
license: MIT
metadata:
  adapted-from: https://github.com/cursor/plugins/tree/main/pstack/skills/figure-it-out
  also-from: https://github.com/cursor/plugins/tree/main/pstack/skills/poteto-mode/playbooks/multi-phase-plan.md
---

# Plan

This is the write path into `vault/` for plans. You own the plan, not the code: do not implement, and stop once it is written and presented.

1. Read `vault/README.md` for the layout and conventions, then `vault/index.md`. Read each touched repo's `decisions.md` and lessons, which are binding constraints, and any plan that overlaps.
2. Frame before exploring. State done as a falsifiable predicate a reviewer can check on the real artifact. Quantify the scope in rough units. Pick the rigor: a one-way door or a wide blast radius gets gates and evidence, a reversible step gets less.
3. Settle open questions by running, not asking. A question about behavior, timing, layout, or whether an API works is answered by a throwaway prototype, and the plan records the result. Ask the user only a product or preference call no run can settle, with concrete options. When nobody can answer, write the assumption into the plan.
4. Explore the affected code, its conventions, tests, and verification commands, delegating to subagents where available. Each checkout's own `CLAUDE.md` or `AGENTS.md` applies.
5. Write `overview.md` per the README's numbering, naming, `Branch:`, and `Status:` conventions; the branch is what `scripts/checkout <owner/repo>@<name> --new` will create. Scrutinize each part against these, in order:
   - Every step is a box naming its evidence, per the README, ordered riskiest unknown first with scaffold and verification before features, and each lands on its own.
   - Every alternative weighed is named with the reason it lost. No reason, no mention.
   - A design plan writes the caller's usage first and derives the shape from it; the shape never overrides the usage.
   - Risks are listed with the step they land in. Nothing in it is code: name types, functions, and files for a senior engineer, not a compiler.
   - Anything the code or an existing note already says is a link to it, not a restatement.
6. Add the checkbox to `vault/index.md` and commit. Add files beside the overview only when they have content.

End with a short summary: the predicate, the steps, the branch, what the prototypes settled and what stays open, and where the overview is.
