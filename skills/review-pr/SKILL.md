---
name: review-pr
description: >
  Review a pull request in this ReactJS + TypeScript codebase and produce a
  prioritized, actionable review. Use this whenever the user says "review this
  PR", "review the pull request", "look over PR #123", "what do you think of
  this PR", or asks for code review on a branch or diff — even if they don't say
  the word "review" explicitly. Checks correctness, React hooks/render
  correctness, TypeScript soundness, accessibility, performance, security, and
  test coverage, then reports findings grouped by severity.
allowed-tools: Bash(gh pr:*), Bash(gh api:*), Bash(git diff:*), Bash(git log:*), Bash(git fetch:*), Bash(git merge-base:*), Read, Grep, Glob, TodoWrite
argument-hint: "[PR number or URL — optional; defaults to the PR for the current branch]"
---

# Review PR

Produce a focused, prioritized review. A good review catches real problems and
respects the author's time — it does not drown a few important issues in a sea
of nitpicks. Be specific: point to the exact file and line, explain *why*
something matters, and suggest a concrete fix.

## 1. Load the diff

- If given a PR number/URL: `gh pr view <n> --json title,body,headRefName,baseRefName,files` and `gh pr diff <n>`.
- If nothing is given: review the current branch against its base. Find the base
  with `git merge-base HEAD origin/<default-branch>` (detect the default branch;
  it's usually `main`, sometimes `master` or `develop`), then `git diff <base>...HEAD`.
- Read the PR description to understand intent. Review against *what it claims to
  do*, not just what the code does.
- For changed files where the diff lacks context, `Read` the full file so you
  review the change in its real surroundings.

## 2. Review checklist

Work through these. Skip categories that don't apply rather than padding.

**Correctness & logic**
- Does it do what the PR says? Edge cases: empty, loading, error, null/undefined, boundary values.
- Off-by-one, inverted conditions, unhandled promise rejections, swallowed errors.

**React-specific**
- Hooks rules: no conditional/looped hooks; complete and correct dependency arrays in `useEffect`/`useMemo`/`useCallback`.
- Effects with side effects clean up (subscriptions, timers, listeners, aborts).
- State that should be *derived* during render instead of synced via an effect.
- Stable, meaningful `key` props on lists (not array index when items reorder).
- Avoidable re-renders (new object/array/function identities passed to memoized children); but flag premature memoization too.
- No state mutation; immutable updates.

**TypeScript**
- No new `any` or unsafe `as` casts; prefer `unknown` + narrowing. No non-null `!` hiding a real nullable.
- Props/return types are explicit where it aids readability; discriminated unions for variant props.
- Public APIs are typed; no implicit `any` leaking across module boundaries.

**Accessibility**
- Semantic elements over click-handlered `div`s; interactive elements are focusable and keyboard-operable.
- Labels for inputs, `alt` for images, sensible ARIA only where native semantics fall short.

**Performance**
- Large lists virtualized where relevant; no O(n²) work in render; no heavy sync work blocking paint.
- Network: no waterfalls that should be parallel; appropriate caching/invalidation.

**Security**
- `dangerouslySetInnerHTML` only with sanitized input; no untrusted HTML/URLs.
- No secrets, tokens, or keys committed; no `eval`/dynamic code from user input.

**Tests & docs**
- New logic has tests covering happy path + key edge cases.
- Public behavior changes are reflected in docs/types/Storybook if the repo uses them.

**Consistency**
- Matches existing conventions in neighboring files (naming, structure, imports, styling approach). Grep for similar patterns before claiming something is "wrong" — it may be the house style.

## 3. Report format

Group every finding by severity so the author knows what blocks merge:

```
## Review: <PR title>

**Summary** — 2–3 sentences: what the PR does and your overall read.

### 🔴 Blocking
- `path/to/file.tsx:42` — <issue>. <why it matters>. <suggested fix>

### 🟡 Should fix
- ...

### 🟢 Nits / optional
- ...

**Tests**: <present & adequate? gaps?>
**Verdict**: Approve / Approve with comments / Request changes
```

If there are no blocking issues, say so plainly. Praise something genuine when
it's there — good reviews are not only criticism. Do **not** post the review to
GitHub or change any code; output it here for the user to act on.
