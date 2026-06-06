# Base standards

These are the baseline agent directives that apply to every project, regardless
of language or framework. Project-specific standards are layered on top in
later files (`01-typescript.md`, `02-spanish-ui.md`, etc.).

## Communication

- Be concise. Prefer one clear sentence over a paragraph.
- Match the user's language. If the user writes in Spanish, reply in Spanish.
- When referencing code, use `file_path:line_number` so the user can jump to it.
- Don't narrate internal deliberation. State results and decisions directly.

## Scope discipline

- Don't add features, refactors, or abstractions beyond what the task requires.
- A bug fix doesn't need surrounding cleanup. A one-shot script doesn't need a
  helper. Three similar lines are better than a premature abstraction.
- Don't design for hypothetical future requirements.
- No half-finished implementations. If a task can't be completed, say so.

## Safety

- Confirm before destructive or hard-to-reverse actions: deleting files or
  branches, force-pushing, dropping tables, rewriting published history.
- Investigate unfamiliar files or branches before overwriting them — they may
  be the user's in-progress work.
- Never skip pre-commit hooks (`--no-verify`) or signing unless explicitly asked.
- Never commit secrets (`.env`, credentials, tokens). Warn if asked to.

## Comments and docs

- Default to writing **no comments**. Only add one when the *why* is non-obvious:
  a hidden constraint, a subtle invariant, a workaround for a specific bug.
- Don't explain *what* the code does — names should do that.
- Don't reference the current task or fix in comments ("added for issue #123") —
  that belongs in the commit message, not the file.
- Don't create planning, decision, or analysis docs unless asked.

## Verification

- For UI changes, run the dev server and verify in a browser. Type checks and
  test suites verify *code correctness*, not *feature correctness*.
- If you can't actually test something, say so explicitly rather than implying
  success.
