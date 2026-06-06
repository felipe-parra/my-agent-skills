---
name: customize-claude
description: >
  Translate project knowledge — conventions, recurring procedures, guardrails,
  or "we always do X" observations — into the right Claude Code primitive:
  a **skill**, a **slash command**, or a **hook**. Use this whenever the user
  says "customize this for my project", "turn this into a skill", "make a
  slash command for X", "add a hook that does Y", "I keep doing X manually,
  let's automate it", "encode this convention", "extend Claude Code for this
  repo", or just after running install.sh on a new project. Picks the right
  primitive via a decision matrix, picks the right scope (project vs user),
  scaffolds the file, and verifies it loads.
allowed-tools: Bash(ls:*), Bash(cat:*), Bash(test:*), Bash(git rev-parse:*), Bash(find:*), Read, Grep, Glob, Write, Edit, TodoWrite
argument-hint: "[optional: the knowledge to encode — a doc path, a one-liner, or a hint at what to automate]"
---

# Customize Claude Code for this project

You installed the base skills and standards. Now this project needs its own
layer: stack-specific conventions, recurring tasks, guardrails. This skill
takes a fuzzy input ("we always run `pnpm typecheck` before pushing", "all our
API errors return `{ code, message }`", "block edits to `schema.sql` without
review") and turns it into the *right kind* of Claude Code customization.

Three primitives are available. Picking the wrong one is the most common
mistake — the decision matrix below exists to prevent it.

## 1. Gather the knowledge

Resolve the argument into something concrete:

- **Inline note** ("we use Zustand, not Redux") → use directly.
- **Doc path** → `Read` the file.
- **Nothing given** → ask: *"What convention, recurring task, or guardrail do you want to encode?"* Then probe for one of:
  - A procedure the user repeats by hand.
  - A convention the agent keeps violating.
  - A guardrail that should fire automatically.

Before continuing, restate the knowledge in one sentence so the user can
correct you. Example:
> **Encoding:** Before any `git push`, run `pnpm typecheck && pnpm test`.

## 2. Classify: skill, command, or hook

Walk this matrix in order. The first match wins.

| Question | If yes → |
|---|---|
| Must this run **automatically** on a Claude Code event (tool call, file edit, session start, prompt submit)? | **Hook** |
| Must this **block** an action from happening (e.g. prevent a write, deny a command)? | **Hook** (`PreToolUse`) |
| Will the **user** type this explicitly to invoke it (`/foo`)? | **Slash command** |
| Is it a **fixed procedure** with no branching — same steps every time? | **Slash command** |
| Should the **agent decide** when to invoke it, based on what the user says or asks for? | **Skill** |
| Is it a **workflow with judgment** — multiple decision points, branching, "do this *unless* that"? | **Skill** |

Quick translation of common asks:

- *"We always run typecheck + tests before push"* → **Hook** (`PreToolUse` matching `git push`)
- *"I want to run our release script with one keystroke"* → **Slash command** (`/release`)
- *"When the user asks to add a feature, follow our scaffolding pattern"* → **Skill** with trigger phrases
- *"Block edits to `prisma/schema.prisma`"* → **Hook** (`PreToolUse` matching `Edit` on that path)
- *"When I say 'ship it', do the release flow"* → **Skill** (it's intent-based, not literal command-based)

If two answers fit, prefer the more conservative one: **hook > command > skill**.
Hooks are deterministic; commands are explicit; skills are inferred.

## 3. Pick the scope

| Scope | Path | When |
|---|---|---|
| **Project** | `.claude/` (commit to repo) | Convention specific to this stack/repo — what every contributor needs. |
| **User** | `~/.claude/` | Personal preference — what *you* want everywhere, but isn't this project's concern. |
| **Local** | `.claude/settings.local.json` (gitignored) | Per-machine tweaks, secrets, experimental hooks. |

Default to **project scope** for anything that belongs to the repo's behavior.
Use **user scope** when the user says "for me" or "everywhere I work."

Run `git rev-parse --show-toplevel` to anchor at the repo root so files land
in the right place.

## 4. Scaffold the file

Use the template that matches the primitive. Replace `<placeholders>` with
real values; remove sections that don't apply.

### Template: Skill

`.claude/skills/<skill-name>/SKILL.md`:

```markdown
---
name: <skill-name>
description: >
  <one or two sentence description ending with concrete trigger phrases:
  "Use this whenever the user says X, Y, or Z, or asks for ...">
allowed-tools: <smallest set of tools that actually do the work — e.g. Read, Grep, Glob, Bash(pnpm test:*)>
argument-hint: "[optional argument hint]"
---

# <Title>

<One paragraph stating what this skill does and what it deliberately does NOT do.>

## 1. <First step>
...

## 2. <Second step>
...

## What this skill does NOT do
- <Explicit non-goals so the agent doesn't overreach.>
```

Skill checklist:
- [ ] Trigger phrases are *concrete* user wordings, not abstract topics.
- [ ] `allowed-tools` is the minimum set — broad tool grants weaken the skill.
- [ ] Steps are short, numbered, and verifiable.
- [ ] There's a "does NOT do" section to prevent scope creep.

### Template: Slash command

`.claude/commands/<command-name>.md`:

```markdown
---
description: <one-line description shown in the slash menu>
argument-hint: "[optional arg hint]"
allowed-tools: <minimum tools — e.g. Bash(pnpm:*), Read>
---

<The literal prompt that runs when the user types /command-name.

Use $ARGUMENTS to reference any arguments passed after the command. Keep this
focused — slash commands are for fixed procedures, not branching workflows.>
```

Slash-command checklist:
- [ ] The prompt body is *exactly* what you want the agent to do — no ambiguity.
- [ ] Arguments (if any) are referenced with `$ARGUMENTS`.
- [ ] If the procedure has branching ("do X unless Y"), it should be a skill instead.

### Template: Hook

Hooks live in `.claude/settings.json` (project) or `~/.claude/settings.json`
(user). Read the existing file first with `Read`; merge — never overwrite.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'about to run a bash command' >&2"
          }
        ]
      }
    ]
  }
}
```

Hook events and when each fires:

| Event | Fires | Common use |
|---|---|---|
| `SessionStart` | New session begins | Inject project context, warn about uncommitted state |
| `UserPromptSubmit` | User submits a prompt | Pre-process input, inject context |
| `PreToolUse` | Before a tool runs (can **block**) | Guardrails, policy enforcement |
| `PostToolUse` | After a tool runs | Format-on-edit, lint-after-write, telemetry |
| `Stop` | Agent finishes a turn | Cleanup, notifications |

Hook checklist:
- [ ] Read the existing `settings.json` first — don't clobber other hooks.
- [ ] The `matcher` is specific (`Bash(git push:*)`, not `Bash`) for blocking hooks.
- [ ] Blocking hooks exit non-zero with a message on stderr — that's how they
      communicate to the agent.
- [ ] Project-scoped hooks go in `.claude/settings.json`; secrets and per-machine
      experiments go in `.claude/settings.local.json` (gitignored).

## 5. Verify

After scaffolding, prove it loads:

- **Skill** — `ls .claude/skills/<skill-name>/SKILL.md` and confirm the frontmatter
  block parses (starts and ends with `---`, has `name:` and `description:`). Tell
  the user a trigger phrase to try.
- **Slash command** — `ls .claude/commands/<command-name>.md`. Tell the user to
  type `/<command-name>` to invoke it.
- **Hook** — `Read` the modified `settings.json` and confirm valid JSON. For
  blocking hooks, suggest a one-line test (e.g. "try to run `git push` — it
  should be intercepted").

Then add a short note to the changelog or commit message describing what was
encoded and why — future-you will not remember.

## What this skill does NOT do

- Does not install the file in user scope without being asked. Project scope
  is the default.
- Does not edit `settings.json` without first reading the existing file.
- Does not invent trigger phrases for skills — they come from how the user
  actually talks about the task.
- Does not bundle multiple unrelated customizations into one file. One concept
  per skill / command / hook.
