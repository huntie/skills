---
name: submit
description: Promote the current branch's local commit to a GitHub pull request, moving the commit message body into the PR description (under the repo's PR template when there is one) and leaving the commit with its title alone. Takes "draft" to open the PR as a draft. Use only when the user invokes /submit or asks to submit the branch as a PR; for several commits as stacked PRs, use submit-stack.
---

# Submit

Submitting publishes a commit prepared with the commit-msg skill as a pull request. From then on the PR description holds the message body and is its source of truth, edited on GitHub as often as not, and the commit keeps only its title, so the two never drift apart.

Takes an optional `draft` argument.

## Preconditions

- Git, or Sapling where git config has `devroot.vcs=sapling`, which follows its own section below.
- The branch holds exactly one commit over its base, `origin/<default branch>` or the upstream's on a fork; in Sapling, `sl log -r 'draft() & ::.'` lists one commit. None: stop. Several: stop and offer squash, or submit-stack for one PR per commit.
- The branch is a task branch, never the default branch.
- Nothing here touches the working tree or the index.

The user's invocation is the confirmation for pushing and opening the PR; never push anywhere else.

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

1. Read the PR fresh: `gh pr view <number> --repo <repo> --json title,body`.
2. A body on the commit again is a revised summary: save the message as in a first submit, write it into the summary section of the fresh description, leaving the rest as the user has it, and drop it from the commit.
3. Push with `git push --force-with-lease origin <branch>`.
4. Update only what changed: `gh pr edit --title` when the commit's title differs from the PR's, and `--body-file` only for a revised summary. Otherwise the description stays exactly as it is on GitHub.
5. `draft` on a resubmit marks an open PR as a draft (`gh pr ready --undo`); leave the state alone without it.
6. Resolve the review threads the push addressed: `scripts/pr-resolve-threads <checkout> --pr <number>` resolves the unresolved threads GitHub marked outdated by the push, and `--all` every unresolved one, for a push that addressed comments on lines that did not change. Leave a thread open when its comment was not acted on, such as a design question; `--dry-run` shows the split first.

### Sapling

The same steps and the same description, with `sl pr submit` doing the push and the PR creation. The checkout's sl config, set by `scripts/checkout`, pushes each PR from a branch of its own (`pr<number>`) and keeps the PR's title and description on every later submit.

- Read the message with `sl log -r . -T '{desc}'`, and drop the body with `sl metaedit -r . -m "<title>"`.
- Push and open with `sl pr submit`, plus `--draft` for `draft`. It opens the PR with the commit's title and, when it finds one, the raw template as the description; it looks only for lowercase `pull_request_template.md`, so replace the description at once with `gh pr edit <number> --body-file <file>`, reading the number from `scripts/pr-info <checkout>`.
- Resubmit with `sl pr submit`. It leaves the title and description alone, so a changed title goes through `gh pr edit --title`, and a revised summary through `--body-file`, as above.

## The description

Find the repo's PR template in GitHub's usual locations, case ignored, including `.github/PULL_REQUEST_TEMPLATE/`; with several, pick the one matching the change, or the default.

- **No template:** The body goes under `## Summary`, and its test plan, when it has one, under `## Test Plan`.
- **Template:** Put the body under the heading closest to a summary, in order of preference: Summary, Description, Motivation, What. Replace that section's guidance comment rather than keeping it beside the body. When no section is an obvious fit, add `## Summary` above the template's sections and put the body there.
- **Sections the body already carries:** When the body ends in a trailer that a template section exists to collect, drop that section and keep the trailer inside the body. In react-native, commit-msg's `Changelog: …` line is that trailer, so the `## Changelog:` section and its comment go, and the whole body, `Changelog:` line included, sits under `## Summary:`.
- **Test Plan:** Fill it from what the session verified, per the test-plan skill: commands, outcomes, and screenshots laid out as it says, with every image a capped `<img>`, whatever the template says about layout. Claim only what was checked. When the session verified nothing, keep the template's prompt and say so in the report.
- **Screenshots:** Upload with `scripts/gh-upload --repo <the repo the PR opens on>`, since an attachment shows only on a PR of the repository it was uploaded to and the script defaults to `origin`, the wrong one in a fork layout. When the token cannot push there, `--release` on `origin` gives public URLs at once.
- **Every other section:** Keep it as the template has it.

Keep the body's Markdown as written; commit-msg already writes it for a reviewer.

## Reporting

The PR URL and its state, the commit's new hash and title, and where the saved message lives. On a first submit, the template sections filled and dropped; on a resubmit, what was pushed and which PR fields changed.
