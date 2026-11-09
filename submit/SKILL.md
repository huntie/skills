---
name: submit
description: Promote the current branch's local commit to a GitHub pull request, moving the commit message body into the PR description (under the repo's PR template when there is one) and leaving the commit with its title alone. Takes "draft" to open the PR as a draft. Use only when the user invokes /submit or asks to submit the branch as a PR; for several commits as stacked PRs, use submit-stack.
license: MIT
---

# Submit

Submitting publishes a commit prepared with the commit-msg skill as a pull request. From then on the PR description holds the message body and is its source of truth, edited on GitHub as often as not, and the commit keeps only its title, so the two never drift apart.

Takes an optional `draft` argument.

## Preconditions

- Git, or Sapling where git config has `devroot.vcs=sapling`, which follows its own section below.
- The branch holds exactly one commit over its base, `origin/<default branch>` or the upstream's on a fork; in Sapling, `sl log -r 'draft() & ::.'` lists one commit. None: stop. Several: stop and offer to squash them into one, or submit-stack for one PR per commit.
- The branch is a task branch, never the default branch. In Sapling on a shared repo, where `origin` is canonical, there is no named branch: the commit is a draft on top of the default branch, and `scripts/sl-pr-submit` pushes it as `pr<number>`. Never push a named branch for it.
- Nothing here touches the working tree or the index.

The user's invocation is the confirmation for pushing and opening the PR; never push anywhere else. It does not cover comments: show the full text of any comment for the PR, such as a reply to a reviewer, and post it only once the user approves it.

## Where the PR goes

- Fork layout, where `upstream` exists: push to `origin`, and open the PR on the upstream repo with `--repo <upstream owner/repo> --head <origin owner>:<branch>`.
- Shared repo, where `origin` is canonical: push to `origin` and open the PR there.
- Base: the target repo's default branch, from `gh repo view <repo> --json defaultBranchRef`.

## Steps

Check the branch first: `scripts/pr-info <checkout>` from the devroot workspace. It prints nothing for a branch with no PR, and otherwise the PR, its state, and `in-sync` or `unsent`. A merged or closed PR means the branch is done: stop and say so.

### First submit

1. Read the message: `git log -1 --format=%B`. The first line is the title, the rest is the body. An empty body is fine; the PR description then holds only what the template needs.
2. Save the full message to `$(git rev-parse --git-dir)/submit/<branch>.md` before anything moves, so a failure later never loses it.
3. Drop the body from the commit: `git commit --amend --only -m "<title>"`. `--only` rewrites the message without taking in anything staged.
4. Build the description (below) into `$(git rev-parse --git-dir)/submit/<branch>-body.md`.
5. Push: `git push --force-with-lease origin <branch>`, since the rewrite changed the commit.
6. Open the PR: `gh pr create --title "<title>" --body-file <file> --base <base> [--repo … --head …] [--draft]`.

### Resubmit

The PR description on GitHub is the source of truth, and the user may have edited it since.

1. Read the PR's title fresh: `gh pr view <number> --repo <repo> --json title`.
2. A body on the commit again is a revised summary: save the message as in a first submit, write it into the summary section of the description as [Editing a submitted description](#editing-a-submitted-description) says, and drop it from the commit.
3. Push with `git push --force-with-lease origin <branch>`.
4. Update only what changed: `gh pr edit --title` when the commit's title differs from the PR's, and `--body-file` only for a revised summary. Otherwise the description stays exactly as it is on GitHub.
5. `draft` on a resubmit marks an open PR as a draft (`gh pr ready --undo`); leave the state alone without it.
6. When the push answers review comments, resolve the review threads it addressed, and leave a thread open when its comment was not acted on, such as a design question.

### Editing a submitted description

Every write to a submitted PR's description, whether a revised summary, a stripped footer or a section the user asked to change, starts from the body on GitHub at that moment, since the user may be editing it there at the same time:

1. Read the body into a new file with `gh api repos/<owner>/<repo>/pulls/<number> --jq .body`. Never edit from an earlier read or a saved copy: the `submit/` files under the git dir hold the message and body as first submitted, and are never a source for a later edit.
2. Change only the sections asked for, and keep the user's wording everywhere else.
3. Check before writing: each replacement target matched exactly once, and a diff of the new file against the fresh read touches only those sections. Chain the write after the check, `<check> && gh pr edit <number> --repo <repo> --body-file <file>`, so a failed check blocks it. On a failure, stop and read the body again; never fall back to an older file.

An overwritten description can be recovered from the PR's edit history: the GraphQL `userContentEdits(first: N)` connection on the pull request holds each earlier version in its `diff` field, with CRLF line endings to strip.

### Sapling

The same steps and the same description, with `scripts/sl-pr-submit` doing the push and the PR creation. It runs `sl pr submit`, which pushes every draft under the working copy, after pulling and refusing a draft that is not yours; never run `sl pr submit` directly. The checkout's sl config, set by `scripts/checkout`, pushes each PR from a branch of its own (`pr<number>`) and keeps the PR's title and description on every later submit.

- Read the message with `sl log -r . -T '{desc}'`, and drop the body with `sl metaedit -r . -m "<title>"`.
- Push and open with `scripts/sl-pr-submit <checkout>`, plus `--draft` for `draft`. It opens the PR with the commit's title and, when it finds one, the raw template as the description; it looks only for lowercase `pull_request_template.md`, so replace the description at once with `gh pr edit <number> --body-file <file>`, reading the number from `scripts/pr-info <checkout>`.
- Before a resubmit, check that `scripts/pr-info <checkout> --rev <node>` names the PR. The link is local to one machine, so a stack handed over from another machine or rebased since has none, and `sl pr submit` then opens a duplicate PR. Find the right commit first: the visible successor of the PR's remote head (`sl log -r 'successors(<head>)'`), confirmed by its own diff matching the head's. Hide a sibling copy with the same content from an older chain, then `sl pr link -r <node> <PR URL>` before submitting.
- Resubmit with `scripts/sl-pr-submit <checkout>`. It leaves the title and description alone, so a changed title goes through `gh pr edit --title`, and a revised summary through `--body-file`, as above.
- A commit in a stack of two or more PRs is the exception when resubmitted from any commit but the bottom one: that push rewrites the description of every open PR from the bottom up to it with Sapling's stack footer appended, a `---` line, the `[//]: # (BEGIN SAPLING FOOTER)` marker and one bullet per PR. The submit-stack skill keeps those descriptions footer-free, so after such a push strip the footer from each body as [Editing a submitted description](#editing-a-submitted-description) says, changing nothing else.

## The description

Find the repo's PR template in GitHub's usual locations, case ignored, including `.github/PULL_REQUEST_TEMPLATE/`; with several, pick the one matching the change, or the default. When the commit-msg skill gives a layout for the repo's description, that layout overrides the rules below and the template's own guidance.

- **No template:** The body goes under `## Summary`, and its test plan, when it has one, under `## Test Plan`.
- **Template:** Put the body under the heading closest to a summary, in order of preference: Summary, Description, Motivation, What. A heading keeps its text but drops a trailing colon. When no section is an obvious fit, add `## Summary` above the template's sections and put the body there.
- **HTML comments:** Delete every `<!-- … -->` comment in the template, then collapse the blank lines left behind so no run of more than one remains. They are guidance for the author, which this skill has already read and followed, and a squash merge copies whatever survives into the landed commit message.
- **Sections the body already carries:** When the body ends in a trailer that a template section exists to collect, drop that section and keep the trailer inside the body.
- **No attribution trailer:** The description carries no `Generated with Claude Code` line or session link, whatever the harness suggests; the commit's `Co-Authored-By` trailer is unaffected.
- **Test Plan:** Fill it from what the session verified, per the test-plan skill: commands, outcomes, and screenshots laid out as it says, with every image a capped `<img>`, whatever the template says about layout. Claim only what was checked. When the session verified nothing, leave the section empty under its heading and say so in the report.
- **Screenshots:** Upload with `scripts/gh-upload --repo <the repo the PR opens on>`, since an attachment shows only on a PR of the repository it was uploaded to and the script defaults to `origin`, the wrong one in a fork layout. When the token cannot push there, `--release` on `origin` gives public URLs at once.
- **Every other section:** Keep it as the template has it, less its comments.

Keep the body's Markdown as written; commit-msg already writes it for a reviewer.

### React Native

The template's headings are `## Summary:`, `## Changelog:` and `## Test Plan:`, and the colons go: `## Summary` and `## Test Plan`. The `## Changelog:` section and its comment go too, since commit-msg's `Changelog: …` trailer is what it exists to collect, and the whole body, that line included, sits under `## Summary`.

## Reporting

The PR URL and its state, the commit's new hash and title, and where the saved message lives. On a first submit, the template sections filled and dropped; on a resubmit, what was pushed and which PR fields changed.
