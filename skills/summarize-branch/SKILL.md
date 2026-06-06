---
name: summarize-branch
description: >
  Summarize the current git branch — what it changes versus its base branch, the
  shape and risk of the diff, and a ready-to-use description. Use this whenever
  the user says "summarize the current branch", "what's on this branch", "what
  have I changed", "summarize my work", or "draft a PR description for this
  branch". Read-only: it inspects history and diffs but never modifies anything.
allowed-tools: Bash(git diff:*), Bash(git log:*), Bash(git status:*), Bash(git merge-base:*), Bash(git rev-parse:*), Bash(git branch:*), Read, Grep, Glob
argument-hint: "[base branch — optional; auto-detected, usually main]"
---

# Summarize Current Branch

Answer "what does this branch do and how risky is it to merge?" — the summary
someone needs before reviewing, or that becomes the PR description.

## 1. Locate the branch and its base

- Current branch: `git rev-parse --abbrev-ref HEAD`.
- Determine the base branch: use the one the user names, else detect the default
  (`main`, then `master`, then `develop`). The fork point is
  `git merge-base HEAD <base>`.
- Don't summarize the entire repo history — summarize only this branch's
  contribution: `<merge-base>..HEAD`.

## 2. Gather the change

- Commits on the branch: `git log <base>..HEAD --pretty=format:'%h %s' --no-merges`.
- Shape of the diff: `git diff --stat <base>...HEAD` (files touched, insertions/deletions).
- Uncommitted work too if relevant: `git status --short` — note if there are unstaged/uncommitted changes the summary should mention.
- For the substantive files, `Read` enough to describe *what* changed, not just *that* it changed. Group files by area (feature, component, tests, config, docs).

## 3. Write the summary

```
## Branch: <name>  (base: <base>)

**What this does** — 2–4 sentences: the goal of the branch and the approach.

**Changes**
- <area / feature>: <what changed and why>
- <area>: ...

**Stats** — N files, +X / −Y across M commits.

**Tests** — added / updated / none (and whether that's a concern).

**Risk & review focus** — the parts a reviewer should look at hardest
(behavioral changes, migrations, anything touching shared/critical code).

**Open items** — TODOs, uncommitted changes, or follow-ups still pending.
```

Keep it honest: if the branch is large or mixes unrelated concerns, say so (and
note it might be worth splitting). If asked for a PR description, format the
above as a clean PR body (summary + bullets + test notes) ready to paste. Output
the summary here; don't push, commit, or open a PR.
