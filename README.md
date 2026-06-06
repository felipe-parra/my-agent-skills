# my-agent-skills

> A versioned, drift-detectable distribution of Claude Code **skills** and
> **agent standards** — installable into any repo with a single command.

[![CI](https://github.com/felipe-parra/my-agent-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/felipe-parra/my-agent-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Use this template](https://img.shields.io/badge/GitHub-Use_this_template-2ea44f?logo=github)](https://github.com/felipe-parra/my-agent-skills/generate)

Built by [@felipe-parra](https://github.com/felipe-parra). The skills and
standards here are the same ones I use across my own projects — published so
they can be reused, audited, and version-pinned.

---

## Why this exists

Anyone using Claude Code seriously across more than one repo hits the same
three walls:

**1. Every new repo needs the same handful of skills.** Review a PR. Plan a
ticket. Investigate a broken build. Refactor a bloated component. Draft a PR
description from a branch. Generate release notes. You re-explain how you want
each of these done on every new project — until you stop bothering, and the
agent reverts to generic answers that read like they were written by a
stranger.

**2. CLAUDE.md drift.** You write a sharp standards block in one repo. Three
months and four repos later, half the rules are different, half are forgotten,
none are in sync. Copy-pasting `.claude/skills/` between repos rots within a
week. You can't tell which project has the *current* version because nothing
is versioned.

**3. No source of truth for agent behavior.** When a project's setup goes from
"great" to "weirdly inconsistent," there's no `git log` to diff against. Was
it a CLAUDE.md edit? A skill change? A drifted copy? Nobody knows, and the
agent's behavior becomes folklore.

**This repo is the single source of truth.** Skills live here, standards live
here. Each consumer repo runs one command to sync, pinned to a tag for
reproducibility, with CI drift detection so projects can't silently fall
behind. New project? Same install, same baseline behavior. Update a standard?
Every consumer can pull it in one PR.

The other half of the problem — every project also has its own stack-specific
conventions (your test runner, your branch naming, your release flow, the
guardrail you keep meaning to add) — is what [`customize-claude`](#2-customize-for-this-project)
handles. The base layer is shared; the customization layer is per-repo.

---

## Quickstart

The workflow is two steps: **install the base layer**, then **customize for
this project**. The base layer is generic and safe to share across repos; the
customization layer encodes what makes *this* repo special.

### 1. Install the base layer

From inside the target repo:

```bash
curl -fsSL https://raw.githubusercontent.com/felipe-parra/my-agent-skills/main/install.sh | bash -s -- .
```

Then add this single line to the project's `CLAUDE.md`:

```markdown
@AGENTS.md
```

That's it for the base — Claude Code now loads every standard on every turn,
and the skills are available under `.claude/skills/`.

For reproducible setups (recommended in shared repos and CI), pin to a tag:

```bash
REF=v0.1.0 curl -fsSL \
  https://raw.githubusercontent.com/felipe-parra/my-agent-skills/v0.1.0/install.sh \
  | bash -s -- .
```

### 2. Customize for this project

The shared standards know about TypeScript and React in general; they don't
know about *your* stack, conventions, or guardrails. That's what the
**`customize-claude`** skill is for.

Open Claude Code in the project and say something like:

> "Customize this for my project — we use Bun, Drizzle, and we always run
> `bun typecheck` before push. Block edits to `drizzle/schema.ts` outside of
> migration PRs."

The `customize-claude` skill takes that fuzzy input and decides — for each
piece — whether it should become a **skill** (intent-based, agent-invoked), a
**slash command** (explicit, user-invoked), or a **hook** (deterministic,
event-driven). Then it scaffolds the right files in the right scope (project
vs user) and verifies they load.

**Why this matters:** it's the difference between a generic install and an
agent that actually understands your codebase. The base layer is the floor;
the customization layer is what makes the install worth keeping.

Re-run `customize-claude` any time the project picks up a new convention or
recurring task. It's not a one-shot — it's the maintenance loop.

### 3. (Optional) Pin and detect drift

In shared repos, pin to a tag (`REF=v0.1.0`) and add a one-line CI job that
re-runs the installer in a clean tree and fails on diff. See [Detecting drift
in CI](#detecting-drift-in-ci) below.

---

## What's inside

```
my-agent-skills/
├── skills/                    # one folder per Claude Code skill
│   ├── create-tests/
│   ├── customize-claude/      # translate project knowledge → skill / command / hook
│   ├── explain-module/
│   ├── generate-release-notes/
│   ├── investigate-build/
│   ├── publish-changes/
│   ├── refactor-component/
│   ├── review-pr/
│   ├── summarize-branch/
│   └── tackle-issue/
├── standards/                 # concatenated into AGENTS.md, in lex order
│   ├── 00-base.md             # tone, scope discipline, safety, comments
│   ├── 01-typescript.md       # TypeScript + React conventions
│   └── 02-spanish-ui.md       # Spanish-language UI copy rules
├── install.sh                 # the installer
└── .github/workflows/ci.yml   # shellcheck + frontmatter + install smoke test
```

The numeric prefix on standards (`00-`, `01-`, `02-`) gives **deterministic
precedence** when files are concatenated — later files layer on top of earlier
ones, so order is a contract, not an accident of the filesystem.

---

## How it works

The installer does three things:

1. **Fetch** the repo at `REF` — shallow `git clone` by default, falling back
   to a GitHub tarball if `git` isn't installed.
2. **Sync skills** into `.claude/skills/` in the target directory.
3. **Concatenate standards** into a single `AGENTS.md` at the target root,
   prefixed with an auto-generated header that records the exact commit SHA
   that produced it.

Your project's `CLAUDE.md` imports `AGENTS.md` with `@AGENTS.md`, so the
standards are loaded on every Claude Code turn — without the project owning a
copy that can drift.

---

## Design choices

A few decisions worth calling out, because they're the difference between
"works on my machine" and "safe to run unattended in 30 repos":

| Choice | Why |
|---|---|
| **Lex-ordered standards** (`00-`, `01-`, `02-`) | Precedence is part of the API. Adding `03-mobile.md` later can't accidentally reorder existing rules. |
| **Auto-generated header on `AGENTS.md`** | The installer refuses to overwrite an `AGENTS.md` that *doesn't* start with the header — so a hand-edited file is never silently clobbered. `FORCE=1` is the explicit opt-out. |
| **`REF` env var pinning** | Consumers can pin to a tag (`v0.1.0`) or SHA. Same install command, reproducible result across machines and time. |
| **Embed the resolved SHA in the output** | The header in `AGENTS.md` records the *resolved* commit SHA, not just the requested ref. `REF=main` today and `REF=main` next week leave different fingerprints — drift is visible. |
| **git OR tarball fetch** | Works on minimal CI images that don't have `git` installed, without adding a dependency. |
| **Drift detection in CI** | Re-run the installer in CI; non-empty `git diff` means the project has fallen behind upstream. One-line check. |
| **CI validates skill frontmatter** | Every `SKILL.md` must have `name:` matching its folder name. Catches broken skills at PR time, not at user-frustration time. |
| **No `npm install`, no runtime deps** | The installer is plain bash. Nothing to audit, nothing to version, nothing to lock. |

---

## Reference

### Environment variables

| Variable          | Default                                                | Purpose                                           |
|-------------------|--------------------------------------------------------|---------------------------------------------------|
| `STANDARDS_REPO`  | `https://github.com/felipe-parra/my-agent-skills`      | Source repo URL                                   |
| `REF`             | `main`                                                 | Branch, tag, or commit SHA to install             |
| `FORCE`           | `0`                                                    | Set to `1` to overwrite a hand-edited `AGENTS.md` |

### Updating an existing project

Re-run the same install command. The installer overwrites the auto-generated
`AGENTS.md` and refreshes `.claude/skills/`. It refuses to overwrite an
`AGENTS.md` that does *not* have the auto-generated header — set `FORCE=1` if
you really mean to.

### Detecting drift in CI

Run the installer in a clean working tree and check for a non-empty `git diff`:

```bash
- name: Verify agent skills are in sync with upstream
  run: |
    curl -fsSL https://raw.githubusercontent.com/felipe-parra/my-agent-skills/main/install.sh | bash -s -- .
    git diff --exit-code .claude/skills AGENTS.md
```

### Adding a new skill

1. Create `skills/<skill-name>/SKILL.md`.
2. Use the standard Claude Code frontmatter:

   ```markdown
   ---
   name: skill-name
   description: >
     One- or two-sentence description that ends with concrete trigger phrases
     ("Use this whenever the user says X, Y, or Z").
   allowed-tools: Read, Grep, Glob, Bash(git diff:*)
   argument-hint: "[optional argument hint]"
   ---

   # Skill Title

   ...body...
   ```

3. Open a PR. CI validates that:
   - The `name:` frontmatter key matches the directory name.
   - The `description:` key is present.
   - The file has a valid frontmatter block.

### Adding a new standard

1. Pick the next numeric prefix (`03-…`, `04-…`).
2. Write the file with a top-level `# Title` heading and short, scannable
   sections.
3. Open a PR.

Standards should be **directives, not exposition.** Tell the agent what to do
or not do, with a one-line *why* when the rationale isn't obvious.

---

## Two ways to consume this

**Use as a GitHub Template** — click ["Use this template"](https://github.com/felipe-parra/my-agent-skills/generate)
to fork the *structure* without the history. Right for someone starting their
own skills collection from scratch.

**Use as an installable bundle** — the `install.sh` flow above. Right for
adding the skills + standards to an existing project, with version pinning and
drift detection. This is the path most consumers want.

---

## License

MIT — see [`LICENSE`](LICENSE).
