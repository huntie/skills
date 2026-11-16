---
name: commit-msg
description: Generate a commit message for the current changes. Use when committing or when asked to draft a commit message.
license: MIT
---

# Commit messages

Read the diff and write the message for a reviewer who sees only the title and the summary. Commit with `sl` where the repo is a Sapling checkout, and with git everywhere else.

## Title

- Imperative, concise and specific: "Fix retry loop in uploader", not "Fixed bug". Around 72 characters, no full stop, plain text with no Markdown.
- Say what the commit does to the codebase, or to a feature as a whole, with a verb for that kind of change: "Replace root module aliases with standard subpath imports", not "Import through package subpath imports". Never pick an imperative that fits one thing the diff touches and turn the rest of the title into its object, and never narrate what the user now does or sees: "Move webpage URL entry to dialog", not "Ask for webpage URL in a dialog".
- Describe the feature, not the plumbing: "Support axisBehavior on bottom toolbar items", not "Set axisBehavior on bottom toolbar items", which hides that a public prop was added.
- Drop articles that add nothing: "Align EPUBs to add page URL above title", not "Open the EPUB with the page URL above the title".
- No possessives: put the owner in front as a plain noun. "Align send sheet page preview with Recents thumbnails", not "Align send sheets' page preview with Recents thumbnails".
- No first person: never "we", "our" or "us". Name the change itself: "Switch to custom dev client", not "Add our own dev client".
- A bug fix opens "Fix …": "Fix sheet background after dark mode change", not "Restore sheet background".
- Prefix it with the scope the change targets, `scope: Subject`, where the scope answers *where*: a directory, package, subsystem or file. Match the repo's history, `git log --no-merges -20 -- <paths you touched>`, including the capitalization after the prefix. Square brackets are markers before the scope, `[WIP]` or `[RFC]`, or a tuple where the repo uses one, such as React Native's `[General][Fixed]`.

## Summary

Two to four sentences, or an opening sentence and a few short bullets under a **Changes** heading, that say what changed and why. Open with the problem as the code behaves today, or with the change as an imperative; never with a result described as a state. Keep what a reviewer acts on: the motivation, behavior a user or caller can observe changing, a decision they might question, a caveat that affects whether it should land, and cross-references such as `Fixes #123`. Leave out what the diff already shows: renames, moved helpers, changed signatures, the order the work happened in, and test results, which go in the test plan.

Bold headings, each on its own line with a blank line before and after, when the summary has two or more parts:

| Heading | Use |
|---|---|
| Context | The effort shared by every commit in a stack, repeated verbatim in each. Always first. |
| Motivation | Why the change is worth making: the user pain or technical driver. |
| Problem / Root cause / Fix | Bug narratives. Use the subset that fits. |
| This diff | What this commit does, when Context or Motivation sets up a broader effort. |
| Impact | Why the change improves things: the win for users, speed, cost or downstream consumers, with its number where one was measured or estimated. Sits before Changes. |
| Changes | Distinct changes grouped by concern, and the default heading for summary bullets. Omit only when a sentence covers the change. |
| Notes | Caveats, limitations or remaining work. |

Backtick every code symbol: API names, packages, paths, flags. In a stack, Context repeats word for word and only This diff changes.

For a visual change or a clear DevTools comparison, include one before/after screenshot that shows the commit's impact.

## Test plan

Optional, as a `**Test plan**` block after the summary when something was run: what was run and what it proved, per the test-plan skill.

## React Native

In react-native and the repos sharing its PR template, `Changelog:` is a required trailer at the end of the summary: `Changelog: [Internal]` for an internal-only change, else `Changelog: [Category][Type] - <description>`, with the category `General`, `iOS` or `Android`, cased exactly so, and the type `Added`, `Fixed`, `Changed`, `Breaking` or `Deprecated`. Put the entry on the line after `Changelog:`, joined by a Markdown hard break (two trailing spaces). The template's `## Summary` heading hosts the message's own subheadings; keep them under it rather than flattening the body into prose. Don't copy the shape of a landed commit on `main`: Meta's import rewrites it into `Summary:`, `## Changelog:` and `[IOS] [ADDED]`, which contributors never write.

## Committing

Re-read the existing message before overwriting it; the user may have edited it.

```bash
git commit -m "<Title>" -m "<Summary>"
sl commit -l <file>  # the whole message, title on the first line
```

Edit a message with `git commit --amend --only -F <file>`, which leaves staged changes out, or `sl metaedit -r <rev> -l <file>` in a Sapling stack.
