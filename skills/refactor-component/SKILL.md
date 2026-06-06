---
name: refactor-component
description: >
  Refactor a React component (or hook/module) in this TypeScript codebase to
  improve clarity, structure, and maintainability without changing its
  observable behavior. Use this whenever the user says "refactor this
  component", "clean up this file", "this component is too big", "improve this
  code", "extract a hook from this", or "make this more maintainable". Verifies
  behavior is preserved via types and tests before and after.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), TodoWrite
argument-hint: "<path to the component/hook/module to refactor>"
---

# Refactor Component

The contract of a refactor: **behavior stays the same, the code gets better.**
If a change would alter what the user sees or how the API behaves, that's a
feature change — call it out and get agreement first; don't smuggle it into a
"refactor".

## 1. Establish the safety net first

Before touching anything:

- `Read` the component fully and understand its current behavior and public props/return shape.
- Find existing tests (`Grep` for the file name in `*.test.*`). Run them and confirm they pass — this is your before-state baseline.
- If there are **no** tests for non-trivial logic, offer to add a few characterization tests first (you can hand off to the `create-tests` skill). Refactoring untested complex code is risky; say so.
- Note the public surface (props, exported types, return values). These must not change unless the user agreed to it.

## 2. Identify what actually needs improving

Don't refactor for its own sake. Look for real issues:

- **Component doing too much** → extract sub-components or, for logic, a custom hook (`useXxx`).
- **Tangled state** → derive values during render instead of syncing via effects; colocate state with where it's used; lift only as far as needed.
- **Prop drilling** several levels → consider context *only* when it clearly pays off (it trades explicitness for convenience).
- **Repeated logic** → extract a shared util/hook.
- **Effect soup** → split unrelated effects; remove effects that just compute derivable values; ensure correct deps and cleanup.
- **Weak types** → replace `any`/unsafe casts with precise types; model variants with discriminated unions; extract shared types.
- **Naming/readability** → clearer names, earlier returns, smaller functions.

State the plan briefly (a short todo list for multi-step refactors) so the user
can redirect before you start editing.

## 3. Refactor in small, verifiable steps

- Make one cohesive change at a time; keep the working tree in a state where types pass between steps.
- Preserve the public API exactly. If you extract a hook or sub-component, the original component's external behavior and props are unchanged.
- Update imports across the codebase if you move/rename anything (`Grep` for usages first).
- Don't reformat unrelated code or churn the whole file — keep the diff reviewable and focused on the actual improvement.

## 4. Verify behavior is preserved

After each meaningful change, run:

- Type check: `npx tsc --noEmit` (or the repo's `typecheck` script).
- The relevant tests (and add cases if the refactor exposed an untested branch).
- Lint if the repo has it, to stay consistent with house style.

Finish with a short summary: what changed, what stayed the same (especially the
public API), and the verification you ran. If you found a latent bug while
refactoring, report it separately — don't silently "fix" behavior under the
banner of a refactor.
