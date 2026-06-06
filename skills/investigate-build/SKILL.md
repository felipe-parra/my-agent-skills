---
name: investigate-build
description: >
  Investigate and diagnose a failing build, type check, lint, or test run in
  this ReactJS + TypeScript project, find the root cause, and propose a fix. Use
  this whenever the user says "investigate the failing build", "the build is
  broken", "why is CI failing", "fix the build error", "TypeScript won't
  compile", or pastes a build/test error. Reproduces the failure, traces it to
  its real cause, and proposes a minimal fix (without applying destructive
  changes unprompted).
allowed-tools: Read, Grep, Glob, Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(git log:*), Bash(git diff:*), Bash(git bisect:*), TodoWrite
argument-hint: "[the failing command or pasted error — optional]"
---

# Investigate Failing Build

Find the *root* cause, not the first red line. Build output is noisy: the real
error is often above the final summary, and one upstream failure can cascade
into many downstream ones. Read carefully before concluding.

## 1. Reproduce

- Identify the failing command. Prefer what the user gives you or what's in `package.json` scripts / the CI config (`.github/workflows/*`, etc.).
- Run it locally and capture the **full** output, not just the tail.
- If it passes locally but fails in CI, that gap *is* the clue — look at Node version, env vars, `NODE_ENV`, install step (`npm ci` vs `npm install`), case-sensitive imports (CI is usually Linux), or out-of-memory on large builds.

## 2. Classify the failure

Knowing the category points at the cause:

- **TypeScript** (`tsc`/build type errors) → read the error path; a type changed somewhere upstream and broke a consumer. `npx tsc --noEmit` to see all of them at once.
- **Lint** → ESLint rule violation; the message names the rule.
- **Tests** → a behavior or a test assumption broke; run the single failing test in isolation.
- **Bundler** (Vite / webpack / Rollup) → module resolution, missing/extraneous dependency, bad import path/casing, env-specific config, or a transform/plugin error.
- **Dependency / install** → version conflict, peer-dep mismatch, lockfile out of sync, native build step failing.
- **Resource** → OOM (`JavaScript heap out of memory`), timeouts.

## 3. Trace to root cause

- Read the actual error and the file/line it points to. Open the file (`Read`) and surrounding context.
- Distinguish symptom from cause. One missing export or a changed type can produce dozens of errors — fix the source, not each symptom.
- If it's a regression that "used to work", find what changed: `git log --oneline -15`, `git diff` the suspect files, and consider `git bisect` for a stubborn regression across many commits.
- For dependency issues, check the lockfile and `package.json` ranges; confirm the installed version vs the expected one.

## 4. Propose the fix

Report findings clearly:

```
## Build failure: <command>

**Symptom** — the error as it appears.
**Root cause** — what's actually wrong, and how you confirmed it.
**Fix** — the minimal change that addresses the cause (with the exact edit).
**Cascade** — other errors this should resolve, if any.
**Prevent** — optional: a guardrail (type, test, CI step) so it doesn't recur.
```

Apply the fix only if it's small, safe, and clearly correct, then re-run to
confirm green. For anything risky — deleting code, bumping a major dependency
version, changing build/CI config, force-resolving dependencies — present the
plan and **ask before applying**. Verify the build passes before declaring it
fixed.
