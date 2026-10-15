---
name: principle-minimal-comments
description: "Apply when writing or reviewing code and its comments. Most code carries none; write one only where the code is unclear and cannot be made clear, and write it for a reader seeing the final version cold."
license: MIT
---

# Minimal Comments

Most code needs no comment. Names, types and structure carry the meaning; a comment is the fallback for the rare line that cannot be made clear on its own.

**Why:** A comment is a second copy of an intent the code already states, and copies drift. Every comment spent on the obvious taxes the one that matters.

**The rule**

- Inline code has no comments by default: not the step a line takes, the branch a condition guards, the phase a block belongs to.
- Before commenting an unclear line, make it clear: rename, extract, shrink. Comment only what survives that, and say why, not what.
- What earns one: a reason the code cannot show, a workaround for something external, code that does the opposite of what it appears to do, an invariant a caller must keep.
- At most one doc comment per function, type or module, at its head, and only when the name and signature leave a question. Behaviour spanning units lives in the document read before the code, never repeated in it.

**Write for a reader who never saw the alternative**

The comment describes the code as it stands, not the path that produced it. This is where a long session leaks: having just weighed two designs, it explains the one it rejected, or cites a constraint that only ever bound the abandoned path. The reader arrives cold, sees a claim nothing in the file supports, and cannot tell whether it is load-bearing.

- No history: "changed from", "now uses", "no longer", "previously".
- No rejected alternatives, and no rationale that only makes sense against one.
- No figure or limit the final code does not answer to — a budget that governed a design you did not ship is noise once it ships.
- No narration of work: no phase labels, no TODO without an owner. In a script, the record is a log line or an assertion message, not a comment above it.

**Tests**

- Delete it: is anything lost that the code or the doc does not already say?
- Would a rename or an extraction remove the need?
- Does it name something absent from the file — a discarded approach, a limit, another design? Then it leaked from the turn that wrote it.
- Change the behaviour: does more than one place go stale?
