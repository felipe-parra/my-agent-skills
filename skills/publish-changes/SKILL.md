---
name: publish-changes
description: >
  Commit and publish the current working-tree changes as a set of focused,
  logically grouped commits using Conventional Commits messages, then optionally
  push. Use when the user runs /publish-changes or asks to "commit my changes",
  "publish these changes", "commit and push", or "group these into commits". It
  inspects the diff, clusters related files into separate commits, writes
  conventional messages, and confirms before pushing. Set to explicit-invoke
  only because committing is a deliberate, state-changing action.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git log:*), Bash(git restore:*), Bash(git reset:*), Bash(git rev-parse:*), Bash(git branch:*), Read, Grep, TodoWrite
disable-model-invocation: true
argument-hint: "[optional context, e.g. 'split feature and tests; don't push']"
---

# Publish Changes

Turn a working tree full of mixed edits into a clean, readable history: several
**focused commits**, each one coherent and independently reviewable, with
Conventional Commits messages. A good commit history tells the story of *why*
the code changed — it is not one giant "wip" blob.

## 1. Survey what's changed

- `git rev-parse --abbrev-ref HEAD` (know the branch — never commit straight onto a protected branch like `main` without the user's OK).
- `git status --short` and `git diff` (and `git diff --staged`) to see every change, staged and unstaged.
- `Read` the diffs enough to understand each change's *purpose*, not just the filenames.

## 2. Safety checks before staging

- **Secrets**: scan the diff for `.env` files, API keys, tokens, passwords, private keys. If anything looks sensitive, **stop and warn** — do not commit it; suggest `.gitignore` / removal.
- **Junk**: flag accidental inclusions (debug `console.log`, commented-out blocks, `*.log`, build artifacts, `node_modules`). Ask before committing them.
- **Scope creep**: if the changes mix clearly unrelated work, that's a signal to split into multiple commits (next step), not to bundle.

## 3. Group into logical commits

Cluster changed files/hunks by *concern*, so each commit does one thing:

- Feature code for one capability → one commit.
- Tests for that feature → can be the same commit, or its own `test:` commit if the repo prefers.
- Unrelated bug fix → its own `fix:` commit.
- Config/build/tooling → its own `build:`/`chore:`/`ci:` commit.
- Docs → its own `docs:` commit.

Stage each group **intentionally** with explicit paths
(`git add path/a path/b`). **Never** blindly `git add -A` / `git add .` — that
defeats the purpose of grouping and risks committing junk or secrets. (If a
single file legitimately contains two unrelated changes, mention it; the user
can decide whether to `git add -p` to split hunks.)

Sketch the planned commits as a short list and confirm the grouping with the
user before committing if there's any ambiguity.

## 4. Write Conventional Commit messages

Format:

```
type(scope): short imperative subject

optional body explaining what and why (wrap ~72 cols)

optional footer: BREAKING CHANGE: ... / Refs #123 / Closes #456
```

Rules:
- **type** ∈ `feat | fix | docs | style | refactor | perf | test | build | ci | chore | revert`.
- **scope** optional, lowercase, the area touched (e.g. `auth`, `cart`, `api`).
- **subject**: imperative mood ("add", not "added"/"adds"), lowercase start, no trailing period, ≤ ~72 chars.
- **body** when the *why* isn't obvious from the subject. **footer** for breaking changes (`BREAKING CHANGE:` or `feat!:`) and issue references.

Examples:
- `feat(cart): add quantity stepper to line items`
- `fix(auth): prevent token refresh loop on 401`
- `refactor(dashboard): extract useBalances hook from Summary`
- `test(cart): cover empty-cart and max-quantity states`
- `perf(list): memoize row renderer to cut re-renders`
- `chore(deps): bump vite to 5.4.2`
- `docs(readme): document the env vars for local dev`
- Breaking:
  ```
  feat(api)!: return ISO dates instead of epoch millis

  BREAKING CHANGE: `createdAt` is now an ISO 8601 string; update
  consumers that parsed it as a number.
  ```

Match the repo's existing convention if it differs (check `git log --oneline -20`
for the scope style and whether they use bodies).

Commit each group: `git commit -m "subject" -m "body…"` (or a single `-m` when
no body is needed).

## 5. Push — only after confirming

- Show the user the commits you created (`git log --oneline -n <count>`).
- Confirm the target branch and remote, then push: `git push` (or `git push -u origin <branch>` for a new branch).
- **Never force-push** to a shared branch. If a force push seems needed (e.g. after a rebase), explain why and ask first; prefer `--force-with-lease` if the user agrees.
- If the user said not to push (or in their argument), stop after committing and tell them the commits are ready locally.

Finish with a brief recap: the commits made (subjects), what was deliberately
left out (and why), and whether anything was pushed.
