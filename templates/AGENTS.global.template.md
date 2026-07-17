# AGENTS.global Template — Instructions for the Agent

Generate or update the global agent instructions file at `~/AGENTS.md`.

## Steps

**1. Check for existing file**
- If `~/AGENTS.md` already exists, show the current content and ask:
  > "~/AGENTS.md already exists. Overwrite?"
- Wait for explicit confirmation before continuing.

**2. Validate `~/.ai/` is populated**
- Check that `~/.ai/adapters/` and `~/.ai/skills/` exist and are not empty:
  ```bash
  ls ~/.ai/adapters/*.md 2>/dev/null
  ls ~/.ai/skills/*.md 2>/dev/null
  ```
- If either folder is missing or empty → **stop**. Inform the user:
  > "`~/.ai/` is not populated yet. Copy assets from `vibe-playground/` first:"
  > ```bash
  > cp ~/path/to/vibe-playground/adapters/* ~/.ai/adapters/
  > cp ~/path/to/vibe-playground/skills/*   ~/.ai/skills/
  > cp ~/path/to/vibe-playground/templates/* ~/.ai/templates/
  > ```
  > Then ask the agent to run this template again.
- Do not proceed until both folders have content.

**3. Scan available assets**
- Adapters: `ls ~/.ai/adapters/*.md` — collect agent names from filenames.
- Skills: `ls ~/.ai/skills/*.md` — for each file, read its frontmatter `name`, `description`,
  and first heading to extract the trigger pattern.

**4. Generate `~/AGENTS.md`**

Save using the format below. Fill the adapter table and skills index from the scan in step 3 — do NOT hardcode.

**5. Check if a pointer file is needed**
- After writing `~/AGENTS.md`, check if your agent reads a different startup file by default
  (some agents use a different filename as their entry point).
- If yes and that file doesn't exist yet:
  - If you can create a symlink to `~/AGENTS.md`: do so.
  - If not: create the file with content: `👉 Read ~/AGENTS.md`

**6. Generate `~/ALWAYS.md` (optional)**
- If the step 3 scan returned **zero** skills → **skip this entire step**. Do not ask
  the user anything and do not create `~/ALWAYS.md`.
- Otherwise, show the user the full list of skills discovered in step 3.
- Ask:
  > "Which of these skills should be loaded automatically at the start of every session,
  > without needing a trigger phrase? Select any, or none to skip."
- Wait for the user's selection before writing anything.
- If at least one skill is selected → write `~/ALWAYS.md` using the format in the output
  section below, then run step 6b.
- If none selected → skip. Do not create the file.

**6b. Wire the pointer into `~/AGENTS.md`**
- After `~/ALWAYS.md` is written (step 6), ensure `~/AGENTS.md` contains the
  "Always-Active Skills" section that points to it. If `~/AGENTS.md` was already
  written in step 4 without that section, add it now (insert before "## Memory"):
  ```markdown
  ## Always-Active Skills

  If `~/ALWAYS.md` exists, read it now and load every skill listed in it before responding
  to any request this session.
  ```
- This keeps `~/AGENTS.md` and `~/ALWAYS.md` in sync — the pointer must exist or the
  always-active skills are never loaded.

### Tip: Custom install path

This template uses `~/.ai/` by default. <br>
If you've set `$AI_HOME` to another location, update all
`~/.ai/` references after copying:

```bash
grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
```

---
<br>
<br>

# Project Agent Instructions

Reusable AI assets (skills, standards, templates) live in `~/.ai/` — not inside any repo.
Do not create vendor-specific directories (`.agents/`, `.opencode/`, `.kiro/`) for reusable assets.

## When to Read This File

Read this file at the start of every session. After reading:

- **No `./AGENTS.md` in project root** → generate one before starting work:
  tell the user and offer to run `~/.ai/templates/AGENTS.local.template.md`.
- **Stack detected but no `~/.ai/standards/<stack>.md`** → offer to create one:
  > "No standards found for `<stack>`. Generate from `~/.ai/templates/standar.template.md`?"
  Wait for user confirmation before proceeding.
  - If `~/.ai/templates/standar.template.md` exists → run it to generate the file.
  - If not → check if a `vibe-playground/` clone is available and offer to copy the template from there.
  - If neither → ask the user to provide stack conventions or rules to follow, then generate
    `~/.ai/standards/<stack>.md` from that context directly.
- **User request is ambiguous** → check the Skills Index below before asking for clarification.
  A matching skill likely covers it.

## Tool Adapters

Identify which CLI agent you are, then read the matching adapter file:

| Agent | Adapter |
|---|---|
| Freebuff (CodebuffAI) | `~/.ai/adapters/freebuff.md` |
| Kiro CLI | `~/.ai/adapters/kiro.md` |
| OpenCode | `~/.ai/adapters/opencode.md` |
<!-- FILL from ~/.ai/adapters/*.md scan — add rows for any new adapters -->

Read the matching adapter **before** responding — it tells you how to read files and access sessions.

## No adapter for you yet?

Read `~/.ai/templates/adapter.template.md` and create one now.
Save to `~/.ai/adapters/<agent-name>.md`.

## Always-Active Skills

<!-- Include this section ONLY if ~/ALWAYS.md was actually created in step 6/6b.
     Omit it entirely if no always-active skills were selected or if step 3 found zero skills. -->
If `~/ALWAYS.md` exists, read it now and load every skill listed in it before responding
to any request this session.

## Skills Index

<!-- FILL from ~/.ai/skills/*.md scan — one row per file -->
| Skill | Trigger pattern | Description | File |
|---|---|---|---|
| `<name>` | `<example phrases that should load this skill>` | `<one-line description>` | `~/.ai/skills/<name>.md` |

When a user request matches a trigger pattern, read the full skill file before responding.
Do not load skills preemptively — only when the trigger pattern matches.

## Standards

Coding standards live in `~/.ai/standards/<tech-stack>.md`.
Read the matching file when working on a known stack.
If no file exists for the current stack, offer to generate one (see "When to Read This File").

## Memory

No agent here has automatic cross-session memory. `~/.ai/memory/` is for manual reference
files — read on request only, no auto-injection.

---
<br>
<br>

## `~/ALWAYS.md` format *(for step 6 — save to `~/ALWAYS.md`, not to `~/AGENTS.md`)*

```markdown
## Always-Active Skills

Unlike the Skills Index below, these are read proactively at the start
of every session — not gated behind a trigger phrase match.

| Skill | File |
|---|---|
| `<id>` | `~/.ai/skills/<id>.md` |
```

List only the skills the user selected in step 6. One row per skill.
