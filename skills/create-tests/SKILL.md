---
name: create-tests
description: >
  Write tests for a specific file in this ReactJS + TypeScript codebase using
  the project's existing test runner and React Testing Library conventions. Use
  this whenever the user says "create tests for this file", "write tests for
  X.tsx", "add unit tests", "cover this component/hook/util with tests", or
  "this file needs tests" — even if they don't name the framework. Detects
  Vitest vs Jest from the repo, mirrors existing test style, and tests behavior
  rather than implementation.
allowed-tools: Read, Grep, Glob, Write, Edit, Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*)
argument-hint: "<path to the file to test>"
---

# Create Tests

Write tests that would survive a refactor: they assert observable behavior
(what a user sees and does, what a function returns) rather than internal
implementation details (state variable names, call counts of private helpers).

## 1. Understand the project's testing setup — do not assume

Detect, don't guess. Read `package.json` and the lockfile/config:

- **Runner**: look for `vitest` vs `jest` in `devDependencies` and for `vitest.config.*`, `jest.config.*`, or a `jest` key in `package.json`.
  - Vitest → `import { describe, it, expect, vi, beforeEach } from 'vitest'`. Mocks use `vi.fn()` / `vi.mock()`.
  - Jest → globals are usually available (check `globals`/setup). Mocks use `jest.fn()` / `jest.mock()`.
- **DOM/component tooling**: `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom` matchers, and the configured environment (`jsdom`/`happy-dom`).
- **Conventions**: open 1–2 nearby existing test files (`Grep` for `describe(` / `it(` / `test(`). Match their file location (`__tests__/` vs co-located `*.test.tsx`), naming, import style, and helpers/test-utils (custom `render` with providers).

If there is **no** test setup at all, say so and propose adding one (e.g. Vitest +
RTL for a Vite app) before writing tests, rather than inventing config silently.

## 2. Analyze the file under test

`Read` the target file fully. Identify what kind of unit it is and what to cover:

- **Component**: rendering with default and varied props; conditional branches (loading / empty / error / populated); user interactions; accessibility (queryable by role/label).
- **Hook**: use `renderHook` from RTL; test initial value, updates from actions, and cleanup/teardown.
- **Pure function / util**: typical inputs, edge cases, boundaries, and error paths.
- **Module with side effects** (network, storage, timers): mock those boundaries; assert the contract, not the implementation.

## 3. Write the tests

Best-practice patterns to follow:

- **Query by accessibility first**: `getByRole`, `getByLabelText`, `getByText`. Use `data-testid` only as a last resort, and note why.
- **Use `userEvent` over `fireEvent`** for realistic interaction (`await user.click(...)`), and `await screen.findBy...` for async UI.
- **One behavior per test**; descriptive names ("disables submit while saving", not "test 2").
- **Arrange–Act–Assert**, minimal setup, no logic in tests.
- **Mock at the boundary** (the imported module/network), not internal functions. Reset mocks in `beforeEach`.
- **Cover the unhappy paths** — error states and empty states are where bugs hide.
- Keep types intact: tests are TypeScript too; type props/mocks rather than `as any`.

Place the file where the repo's convention dictates and name it accordingly
(`<name>.test.tsx` / `.test.ts`).

## 4. Run and verify

Run only the new test file with the detected runner, e.g.
`npx vitest run path/to/file.test.tsx` or `npx jest path/to/file.test.tsx`
(use the package manager the repo uses). Iterate until green. If a test reveals
what looks like a real bug in the source, **stop and flag it** rather than
writing the test to expect the buggy behavior.

End by summarizing what you covered and any gaps you deliberately left (and why).
