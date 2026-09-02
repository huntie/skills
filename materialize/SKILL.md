---
name: materialize
description: Turn a described change into code and commit it, using the given title and description as the whole specification. Use only when the user invokes /materialize.
---

The user provides a commit title and optional description, separated by a comma or newline. The title says *what* the change is; the description (if any) says *why* or gives additional context.

## Steps

1. Parse the arguments into a commit title and optional description.
2. Analyze the codebase to understand what changes are needed to fulfill the described change. Use the title and description as your only specification — do not ask for clarification.
3. Make the changes.
4. Stage and commit using the parsed title and description. Follow the commit format from the `commit-msg` skill (detect repo type, use the right command), but do **not** generate a new message — use the one provided.
