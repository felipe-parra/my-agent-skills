# my-agent-skills

A shareable set of Claude Code **skills** and **agent standards** that I
(`@felipe-parra`) reuse across projects. One script installs the skills under
`.claude/skills/` and concatenates the standards into an `AGENTS.md` that you
import from `CLAUDE.md`.

> If you want your own version, click **"Use this template"** on GitHub to
> spin up a clean copy with no commit history.

---

## What's inside

```
my-agent-skills/
├── skills/                    # one folder per Claude Code skill
│   ├── create-tests/SKILL.md
│   ├── explain-module/SKILL.md
│   ├── generate-release-notes/SKILL.md
│   ├── investigate-build/SKILL.md
│   ├── publish-changes/SKILL.md
│   ├── refactor-component/SKILL.md
│   ├── review-pr/SKILL.md
│   └── summarize-branch/SKILL.md
├── standards/                 # concatenated into AGENTS.md, in lex order
│   ├── 00-base.md             # baseline: tone, scope, safety, comments
│   ├── 01-typescript.md       # TS + React conventions
│   └── 02-spanish-ui.md       # Spanish-language UI copy rules
├── install.sh                 # the installer (see below)
└── .github/workflows/ci.yml   # lint shell, validate skill frontmatter
```

The numeric prefix (`00-`, `01-`, `02-`) gives **deterministic precedence**
when files are concatenated: later files layer on top of earlier ones.

---

## Install into a project

From inside the target repo:

```bash
curl -fsSL https://raw.githubusercontent.com/felipe-parra/my-agent-skills/main/install.sh | bash -s -- .
```

Or pin to a specific tag (recommended for CI / reproducible setups):

```bash
REF=v0.1.0 curl -fsSL \
  https://raw.githubusercontent.com/felipe-parra/my-agent-skills/v0.1.0/install.sh \
  | bash -s -- .
```

The installer:

1. Fetches the repo at `REF` (shallow clone, or tarball if `git` is missing).
2. Copies `skills/*` into `.claude/skills/` in the target directory.
3. Concatenates `standards/*.md` into `AGENTS.md` at the target root, with a
   header marking it as auto-generated.

Then, in your project's `CLAUDE.md`:

```markdown
# My Project

@AGENTS.md

## Project-specific directives
- (anything that only applies to this repo, layered on top)
```

The `@AGENTS.md` import is what makes Claude Code load the standards on every
turn.

### Environment variables

| Variable          | Default                                                | Purpose                                   |
|-------------------|--------------------------------------------------------|-------------------------------------------|
| `STANDARDS_REPO`  | `https://github.com/felipe-parra/my-agent-skills`      | Source repo URL                           |
| `REF`             | `main`                                                 | Branch, tag, or commit SHA to install     |
| `FORCE`           | `0`                                                    | Set to `1` to overwrite a hand-edited `AGENTS.md` |

---

## Updating

Re-run the same install command. The installer overwrites the auto-generated
`AGENTS.md` and refreshes `.claude/skills/`. It refuses to overwrite an
`AGENTS.md` that does *not* have the auto-generated header — set `FORCE=1` if
you really mean to.

To detect drift in CI (e.g. a project that wandered off-standard), run the
installer in a clean working tree and check for a non-empty `git diff`.

---

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md`.
2. Use the standard Claude Code skill frontmatter:

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

3. Open a PR. CI validates that every `SKILL.md` has the required frontmatter
   keys before merge.

---

## Adding a new standard

1. Pick the next numeric prefix (`03-…`, `04-…`).
2. Write the doc with a top-level `# Title` heading and short, scannable
   sections.
3. Open a PR.

Standards should be **directives**, not exposition. Tell the agent what to do
or not do, with a one-line *why* when the rationale isn't obvious.

---

## Why a template repo + install script (not just one or the other)

- **Template repo** (GitHub "Use this template") is the right entry point for
  someone starting their *own* skills collection — clean history, full control.
- **Install script** is the right entry point for *consuming* this collection
  from another project — pull updates, version-pin, detect drift in CI.

Both modes are supported. Pick whichever fits your workflow.

---

## License

MIT — see [`LICENSE`](LICENSE).
