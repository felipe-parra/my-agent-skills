---
name: tackle-issue
description: >
  Turn a spec, acceptance criteria, GitHub issue, bug report, or task description
  into a concrete, ordered plan for working on it. Use this whenever the user says
  "tackle this issue", "plan this out", "how would you approach #123", "let's
  work on this ticket", "break this down", pastes a spec or acceptance criteria,
  or shares an issue/bug/task URL. Reads the input, maps it to the codebase,
  surfaces ambiguities up-front, and emits a step-by-step plan with risks and
  open questions — does NOT start implementing until the plan is confirmed.
allowed-tools: Bash(gh issue:*), Bash(gh api:*), Bash(git log:*), Bash(git diff:*), Bash(git branch:*), Read, Grep, Glob, TodoWrite
argument-hint: "[issue number, URL, or inline spec — optional; if omitted, ask the user for the input]"
---

# Tackle Issue

Take a fuzzy input (spec, AC, issue, bug, task) and produce a plan crisp enough
that the next step is obvious. A good plan **front-loads ambiguity**: the goal
is to surface every important decision *before* you start changing code, not to
discover them mid-implementation.

This skill stops at "plan ready for review." It does not start editing files.

## 1. Load the input

Resolve the argument into a concrete spec to work from:

- **GitHub issue number or URL** → `gh issue view <n> --json title,body,labels,assignees,state,comments` and read the body plus every comment. Comments often contain the *real* acceptance criteria.
- **Inline spec / pasted text** → use it directly. If it's a screenshot or PDF path, `Read` it.
- **Nothing given** → ask the user: *"What issue, spec, or ticket should I plan against?"* Don't guess.

If the input references other issues (`closes #45`, `blocked by #78`, `see also #92`), open those too — they usually carry the missing context.

## 2. Restate the goal in one sentence

Before doing anything else, write a single sentence that captures what *done*
looks like, in your own words. If you can't, the input is too vague and the
next step is "ask the user," not "start coding."

Example:
> **Goal:** When a logged-out user clicks "Save", redirect them to `/login?next=<path>` instead of silently failing.

## 3. Extract acceptance criteria

Pull out the explicit acceptance criteria as a checklist. If the input doesn't
have them, derive them from the description and flag them as **inferred** so
the user can correct you.

```markdown
**Acceptance criteria**
- [ ] Logged-out click on "Save" redirects to `/login?next=<current-path>`.
- [ ] After login, user lands back on `<current-path>` with their unsaved input intact. *(inferred)*
- [ ] No regression for the logged-in flow.
```

## 4. Map to the codebase

Find where this lives *before* deciding how to change it. For each AC, locate:

- The entry point (file:line) — the component, route, or handler the user interacts with.
- The data path — what reads/writes the state involved.
- Existing tests — `Grep` for related test files; they reveal the contract.

Note anything surprising: dead code paths, recent refactors in the area, feature
flags gating the behavior. `git log --oneline -- <path>` on the most relevant
files shows whether this code is hot or stable.

## 5. Surface ambiguities and risks

List every open question before sketching steps. Categorize:

- **Blocking questions** — can't start without an answer. *Ask the user now.*
- **Assumptions** — you'll proceed unless told otherwise. State them explicitly.
- **Risks** — things that could go sideways: shared state, migrations, third-party APIs, perf-sensitive paths, areas with thin test coverage.

Example:
```markdown
**Blocking**
- Should the redirect preserve query params, or only the pathname?

**Assumptions**
- Auth state is read from `useAuth()` (no SSR auth check needed for this page).
- "Save" is the only entry point that needs this — list views already gate at the row level.

**Risks**
- The form has uncommitted state in a Zustand store. Restoring it after login requires plumbing through `next=` *and* persisting the draft.
```

## 6. Break into ordered steps

Write the plan as a numbered list. Each step should be:

- **Small enough** that it could land as its own commit.
- **Concrete** — name the file, function, or test to touch.
- **Verifiable** — there's something observable after it ("type-checks", "test X passes", "redirect happens in browser").

Group steps into phases when there are natural seams (data layer → UI → tests),
but don't invent phases just to look organized. Three good steps beat ten
ceremonial ones.

```markdown
1. Add `redirectToLogin(nextPath)` helper in `src/lib/auth.ts`. Type-checks.
2. Replace the silent-fail branch in `SaveButton.tsx:42` with `redirectToLogin(pathname)`. Manual: click Save while logged out → land on `/login?next=…`.
3. After login, in `LoginForm.tsx:88`, honor `next` from query params (allow-list paths to prevent open redirect). Test: cypress flow `save-then-login.cy.ts`.
4. Persist draft form state to localStorage on redirect; restore on `next` landing. Test: `useDraftPersistence.test.ts`.
5. Add a regression test that logged-in Save still works unchanged.
```

## 7. Estimate scope and pick a branch strategy

One line each:

- **Effort** — rough t-shirt size (XS / S / M / L). Be honest; if you don't know enough to size it, say so.
- **PR shape** — one PR or stacked? Stacked is rarely worth it for <500 LOC.
- **Branch name** — suggest `feat/<short-slug>` or `fix/<issue-number>-<slug>`.

## 8. Output the plan

Print the full plan in this shape, then stop and wait for the user to confirm,
edit, or redirect. Use `TodoWrite` to mirror the steps as tasks so they survive
into the implementation session.

```markdown
## Plan: <issue title or short slug>

**Goal:** <one sentence>

**Acceptance criteria**
- [ ] ...

**Code map**
- Entry: `path/to/file.tsx:LL`
- Data: `path/to/store.ts`
- Tests: `path/to/file.test.tsx`

**Blocking questions**
- ...

**Assumptions**
- ...

**Risks**
- ...

**Steps**
1. ...
2. ...

**Scope:** S — single PR — branch `fix/123-redirect-on-save`
```

End with: *"Want me to start on step 1, or adjust the plan first?"*

## What this skill does NOT do

- Does not modify files. Planning only.
- Does not open a PR or push a branch.
- Does not run the test suite or dev server.
- Does not fill in missing acceptance criteria silently — inferred ACs are
  always labeled `*(inferred)*` so the user can correct them.
