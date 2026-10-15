---
name: commit-msg
description: Generate a commit message for the current changes. Use when committing or when asked to draft a commit message.
license: MIT
---

# Commit messages

Read the diff and write the message for a reviewer who sees only the title and the summary. Commit with `sl` where the repo is a Sapling checkout, and with git everywhere else.

## Title

- Imperative, concise and specific: "Fix retry loop in uploader", not "Fixed bug". Around 72 characters, no full stop, plain text with no Markdown.
- Prefix it with the scope the change targets, `scope: Subject`, where the scope answers *where*: a directory, package, subsystem or file. Match the repo's history, `git log --no-merges -20 -- <paths you touched>`, including the capitalization after the prefix. Square brackets are markers before the scope, `[WIP]` or `[RFC]`, or a tuple where the repo uses one, such as React Native's `[General][Fixed]`.

## Summary

Two to four sentences, or an opening sentence and a few short bullets, that say what changed and why. Open with the problem as the code behaves today, or with the change as an imperative; never with a result described as a state. Keep what a reviewer acts on: the motivation, behavior a user or caller can observe changing, a decision they might question, a caveat that affects whether it should land, and cross-references such as `Fixes #123`. Leave out what the diff already shows: renames, moved helpers, changed signatures, the order the work happened in, and test results, which go in the test plan.

Bold headings, each on its own line with a blank line before and after, when the summary has two or more parts:

| Heading | Use |
|---|---|
| Context | The effort shared by every commit in a stack, repeated verbatim in each. Always first. |
| Motivation | Why the change is worth making: the user pain or technical driver. |
| Problem / Root cause / Fix | Bug narratives. Use the subset that fits. |
| This diff | What this commit does, when Context or Motivation sets up a broader effort. |
| Changes | Distinct changes grouped by concern. Omit if a sentence covers it. |
| Impact | What this means for users, performance or downstream consumers. |
| Notes | Caveats, limitations or remaining work. |

Backtick every code symbol: API names, packages, paths, flags. In a stack, Context repeats word for word and only This diff changes.

## Test plan

Optional, as a `**Test plan**` block after the summary when something was run: what was run and what it proved, per the test-plan skill.

## React Native

In react-native and the repos sharing its PR template, `Changelog:` is a required trailer at the end of the summary: `Changelog: [Internal]` for an internal-only change, else `Changelog: [Category][Type] - <description>` with the type `Added`, `Fixed`, `Changed`, `Breaking` or `Deprecated`.

## Committing

Re-read the existing message before overwriting it; the user may have edited it.

```bash
git commit -m "<Title>" -m "<Summary>"
sl commit -l <file>  # the whole message, title on the first line
```

Edit a message with `git commit --amend --only -F <file>`, which leaves staged changes out, or `sl metaedit -r <rev> -l <file>` in a Sapling stack.
