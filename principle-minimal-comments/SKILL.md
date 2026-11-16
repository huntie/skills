---
name: principle-minimal-comments
description: "Load before writing, editing or reviewing code in any language, whether or not comments come up, and hold to it over the comment style of the surrounding code. Code carries no comments by default: make it explain itself by renaming, extracting or rewriting. Only a workaround that reads as a mistake without one keeps a comment, written for a reader seeing the final version cold."
license: MIT
---

# Minimal Comments

Code explains itself, or it gets rewritten until it does. A comment is the last resort for the rare line that still reads as a mistake after that: a workaround for something outside the code that no name can carry.

**Why:** A comment is a second copy of an intent the code already states, and copies drift. Every comment spent on the obvious taxes the one that matters, and a reader cannot tell the load-bearing one from the noise.

**The rule**

- No comments by default, inline or at a unit's head: not the step a line takes, the branch a condition guards, the phase a block belongs to, the mechanism a value compensates for.
- The urge to comment means the code is unclear. Fix the code instead: name the value (`const scrollViewInsets = headerHeight + insets.bottom`), extract a function whose name says why, split the expression, restructure the branch. Then drop the comment you were about to write.
- The bar for survival is a workaround for something external, such as a platform bug, a library quirk or a timing constraint, that reads as a mistake and invites a "fix" without the comment. Name the external cause in a line. Nothing else clears it: not an invariant the types or an assertion can carry, not an explanation of layout or arithmetic, not why the code has this shape.
- A workaround that reads as ordinary code gets none: a `jest.mock` placed before a `require` is how tests look, whatever quirk of the toolchain forced the order.
- A named constant or extracted function is the comment. Its name carries the whole explanation, so nothing sits above it: not a doc block, not the sentence the inline comment was going to say. A design value such as a spacing, an offset, a scale or a limit is a number someone chose, and the reason it looks right is taste, not a workaround: `const OFFSET = 16;` and nothing else.
- Doc comments: at most one per exported function, type or module, and only when the name and signature leave a question a caller must answer. `findCheckRun(headSha, name)` leaves none, so `/** The latest check run named name on headSha, or null. */` goes. Behavior spanning units lives in the document read before the code, never repeated in it.

**Surrounding code is not precedent**

The pull to comment rarely comes from the code at hand. It comes from the file around it, a docstring habit, or a language whose tooling expects a doc block on every declaration.

- New and changed lines meet this bar whatever the file does. A file that documents every function is not a style to match: the next function in it gets no doc block unless it passes the tests below.
- Docstring conventions do not override it. JSDoc, Python docstrings, `///` and KDoc are formats for the rare comment that survives, not a quota.
- Leave existing comments alone unless the task touches their lines. A comment on a line you rewrite is held to the bar like a new one.

**Write for a reader who never saw the session**

A comment written mid-task records what the author just learned: the debugging finding, the constraint that ruled out the first attempt, the mechanism behind a bug. That is the author's context, not the reader's. The reader arrives cold, sees ordinary code under a paragraph of backstory, and cannot tell whether it is load-bearing.

- No history: "changed from", "now uses", "no longer", "previously".
- No rejected alternatives, and no rationale that only makes sense against one.
- No debugging findings: how a framework lays out, measures or schedules, learned while fixing this code. If the code depends on it, a name says what the value is for.
- No figure or limit the final code does not answer to.
- No narration of work: no phase labels, no TODO without an owner. In a script, the record is a log line or an assertion message, not a comment above it.

**Tests** — a comment survives only if it passes every one

- Delete it: would a careful reader "fix" the line back into a bug? If not, it goes.
- Would a rename, a named constant or an extracted function say the same? Then do that instead.
- Did the comment move onto the constant or function just extracted, as a doc block? The extraction did its job; the comment goes.
- Does it restate the name, the signature or the next few lines? Then it goes.
- Does it explain how the platform or framework behaves, rather than flag a workaround for it? Then it goes.
- Does it name something absent from the file, such as a discarded approach, a limit or the debugging that found it? Then it leaked from the session that wrote it.
- Reviewing: delete by default, and let a comment earn its way back.

**Before the change is done**

Read back every comment line the diff adds, doc blocks included, and put each through the tests. Most go. Name any that stay, and the external cause each one flags, when reporting the change.
