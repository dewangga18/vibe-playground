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
- Skills: `ls ~/.ai/skills/*.md` — read frontmatter `name`, `description`, first heading.
- If `~/AGENTS.md` already exists (from step 1) and has a `## Skills Index` table, diff the
  newly scanned skills against the ones already recorded there. Skills that appear now but
  weren't recorded = **new skills**.
- Identify the currently active adapter (`~/.ai/adapters/<agent-name>.md`). Check whether that
  file declares a **native always-active mechanism** (steering/hooks path + format).
  - If none / no adapter yet → treat as "no native mechanism", fall back to `~/ALWAYS.md`.

**4. Generate `~/AGENTS.md`**

Save using the format below. Fill the adapter table and skills index from the scan in step 3 — do NOT hardcode.

**5. Check if a pointer file is needed**
- After writing `~/AGENTS.md`, check if your agent reads a different startup file by default
  (some agents use a different filename as their entry point).
- If yes and that file doesn't exist yet:
  - If you can create a symlink to `~/AGENTS.md`: do so.
  - If not: create the file with content: `👉 Read ~/AGENTS.md`

**6. Always-active rules (optional)**

- If step 3 returns zero skills AND this is a first-time build → skip this step entirely.

*First-time build* (no prior `~/AGENTS.md`):
- Show the full list of skills discovered in step 3.
- Ask:
  > "Are there any rules/instructions the agent should apply on EVERY request — not just
  > when a trigger matches? You can pick from the skills above, or give a custom instruction."
- Wait for the user's input. If nothing is selected/provided → skip, don't create any file.

*Regenerate with new skills* (`~/AGENTS.md` already exists, step 3 found new skills):
- Do NOT re-ask about skills that were already recorded.
- For each new skill, the agent analyzes its frontmatter/description/trigger: does it look
  suitable for always-on application (broad, context-independent) vs. narrow/trigger-based?
- If at least one looks suitable → ask the user, showing ONLY those candidates:
  > "The new skill `<name>` looks like it should be applied on every request (because
  > `<reason>`). Add it to the always-active rules?"
- If there are no new skills, or none look suitable → skip silently, don't ask the user.

**Where to write it:**
- If the active adapter (from step 3) has a native mechanism → write/append there, following
  that adapter's declared path & format.
- Otherwise → write/append to `~/ALWAYS.md`. If that file already exists, show its contents
  and confirm append vs. replace first (mirror the pattern used for `~/AGENTS.md` in step 1).

**6b. Wire the pointer into `~/AGENTS.md`**
- Only relevant when step 6's target was the generic `~/ALWAYS.md` fallback. If the rules were
  routed to a native mechanism, do NOT add a "read now" instruction — just an informational
  note, since the native mechanism is already auto-loaded by the tool itself, outside the
  AGENTS.md read path.
- Insert point STAYS: after `## No adapter for you yet?`, before `## Skills Index` — for both
  first build and later patching. (Replaces the old "insert before `## Memory`" instruction,
  which was in the wrong position.)

Generic fallback:<br>
```bash
## Always-Active Skills

If `~/ALWAYS.md` exists, read it now and load every skill listed in it before responding
to any request this session.
```

Native fallback (informational only, not an executable instruction):<br>
```bash
## Always-Active Rules
This agent uses <AgentName>'s native steering/hooks mechanism for always-active rules —
see `~/.ai/adapters/<agent-name>.md` for location and format. Not routed through this file.
```

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
