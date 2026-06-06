# TypeScript & React standards

Applies to any project where TypeScript or React is the primary stack.

## TypeScript

- No `any` without justification. Prefer `unknown` and narrow, or a precise type.
- No non-null assertions (`!`) unless an invariant is documented one line above.
- Use `import type` for type-only imports so they're erased from the bundle.
- Prefer `readonly` and `as const` for data that never mutates after creation.
- Discriminated unions over optional fields when modeling state — they make
  invalid states unrepresentable.
- Don't add runtime validation for values guaranteed by the type system.
  Validate at *system boundaries* (HTTP, user input, env vars) and trust internal
  code afterwards.

## React

- Server Components by default in App Router. Mark with `"use client"` only when
  the component genuinely needs state, effects, or browser-only APIs.
- Keep components small and focused. Extract a hook when a component has more
  than ~3 pieces of related local state.
- Effects are for *synchronizing with external systems* — not for deriving state.
  Derived values belong in render or `useMemo` only when measurement shows it's
  needed.
- Stable keys on lists. Never `key={index}` for lists that can reorder or filter.
- Accessibility: every interactive element needs an accessible name; every form
  control needs a label; respect keyboard navigation and focus order.

## Testing

- Test behavior, not implementation. Don't assert on internal state, classes,
  or component structure if a user-visible outcome would catch the bug.
- Prefer React Testing Library queries by role and accessible name —
  `getByRole('button', { name: /submit/i })` over `getByTestId`.
- Integration tests over unit tests where reasonable. A test that exercises
  three units catches more real bugs than three isolated unit tests.

## Tooling

- Detect the runner before writing tests (Vitest vs Jest vs Node test runner).
  Mirror the project's existing style.
- Prefer the project's existing utilities over introducing new dependencies.
  When adding a dependency, justify it in the PR description.
