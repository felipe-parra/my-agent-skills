---
name: generate-release-notes
description: >
  Generate release notes / a changelog for this project from git history,
  grouped by change type and written for the people who read releases. Use this
  whenever the user says "generate release notes", "write the changelog", "what
  changed since the last release", "prepare release notes for v2.x", or "draft
  the release". Reads conventional-commit history since the last tag, groups and
  rewrites it into human-readable notes, and suggests the next semantic version.
allowed-tools: Bash(git log:*), Bash(git tag:*), Bash(git describe:*), Bash(git diff:*), Bash(gh release:*), Read, Write
argument-hint: "[from-ref..to-ref — optional; defaults to last tag..HEAD]"
---

# Generate Release Notes

Release notes are for humans deciding whether to upgrade and understanding what
changed — not a raw `git log` dump. Translate commit subjects into clear,
user-facing statements and group them so the important stuff (features, fixes,
breaking changes) is easy to find.

## 1. Determine the range

- Default range: from the most recent tag to `HEAD`.
  - Last tag: `git describe --tags --abbrev=0` (if it errors, there are no tags — use the full history or ask the user for a starting point).
  - Then: `git log <last-tag>..HEAD --pretty=format:'%H%x09%s%x09%b' --no-merges`.
- If the user gives an explicit range or two refs, use those.
- Confirm the range you're summarizing in the output header.

## 2. Parse conventional commits

Most subjects follow `type(scope): subject`. Bucket by `type`:

- `feat` → **Features**
- `fix` → **Bug Fixes**
- `perf` → **Performance**
- `refactor` → **Refactors** (include only if user-relevant; otherwise omit or fold into a brief note)
- `docs` → **Documentation**
- `revert` → **Reverts**
- `build`, `ci`, `chore`, `style`, `test` → usually **internal**: omit from user-facing notes, or collapse into a single "Internal/maintenance" line. Don't let them dominate.

**Breaking changes** get their own top section. Detect them via a `!` after the
type/scope (`feat!:`, `feat(api)!:`) **or** a `BREAKING CHANGE:` footer in the
body. Always surface these first and explain the migration.

Commits that don't follow the convention: read the subject and place them by
meaning; don't drop real changes just because they're unformatted.

## 3. Write the notes

Rewrite each entry as a concise, reader-facing line (not the literal commit
subject). Strip the `type(scope):` prefix; lead with what changed for the user.
Reference the PR/commit where useful (`(#123)`), and group by scope within a
section if there are many entries.

```
## <version or range> — <date>

### ⚠️ Breaking Changes
- <what broke and how to migrate>

### ✨ Features
- <capability>, in your own words (#PR)

### 🐛 Bug Fixes
- <what was fixed>

### ⚡ Performance
- ...

### 📝 Documentation
- ...

_Internal: N maintenance/chore/ci commits._
```

## 4. Suggest the next version (SemVer)

Recommend the bump based on the contents:

- Any breaking change → **major**.
- Else any `feat` → **minor**.
- Else only `fix`/`perf`/patches → **patch**.

State the current version (from the last tag) and the suggested next one, and
note it's a suggestion. Ask whether to save the notes to `CHANGELOG.md` (prepend
a new section) or hand them off for a `gh release create`. Don't create the tag
or publish the release yourself unless the user explicitly asks.
