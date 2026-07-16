# AGENTS.global Template — Instructions for the Agent

Generate or update the global agent instructions file at `~/AGENTS.md`.

## Steps

**1. Check for existing file**
- If `~/AGENTS.md` already exists, show the current content and ask:
  > "~/AGENTS.md already exists. Overwrite?"
- Wait for explicit confirmation before continuing.

**2. Scan available assets**
- Adapters: `ls ~/.ai/adapters/*.md` — collect agent names from filenames.
- Skills: `ls ~/.ai/skills/*.md` — collect skill names and read each file's first heading/sentence to infer its trigger and description.

**3. Generate `~/AGENTS.md`**

Save using the format below. Fill the adapter table and skills table from the scan in step 2 — do NOT hardcode.

**4. Check if a pointer file is needed**
- After writing `~/AGENTS.md`, check if your agent requires a different startup file (e.g. `~/CLAUDE.md` for Claude Code / Freebuff).
- If yes and the file doesn't exist:
  - If you can create a symlink: `ln -s ~/AGENTS.md ~/CLAUDE.md`
  - If not: create `~/CLAUDE.md` with content: `👉 Read ~/AGENTS.md`

---
<br>
<br>

# Project Agent Instructions

Reusable AI assets (skills, standards, templates) live in `~/.ai/` — not inside any repo.
Do not create vendor-specific directories (`.agents/`, `.opencode/`, `.kiro/`) for reusable assets.

## Tool Adapters

Identify which CLI agent you are, then read the matching adapter file before responding to any request:

| Agent | Adapter |
|---|---|
| Freebuff (CodebuffAI) | `~/.ai/adapters/freebuff.md` |
| Kiro CLI | `~/.ai/adapters/kiro.md` |
| OpenCode | `~/.ai/adapters/opencode.md` |
<!-- FILL from ~/.ai/adapters/*.md scan — add rows for any new adapters -->

Read the matching adapter **before** responding — not only when the user names a skill.

## No adapter for you yet?

1. Scan `~/.ai/skills/*.md`.
2. Infer each skill's purpose from filename and content.
3. Load the relevant skill using your file-reading tool.
4. Do not hardcode a skill list — new skills need no changes here.

If you're a new agent without an adapter, read `~/.ai/templates/adapter.template.md` and create one.

## Available Skills

<!-- FILL from ~/.ai/skills/*.md scan — one row per file -->
| Skill | Common trigger | Description |
|---|---|---|
|\<skill-name> |\<trigger pattern with example> |\<skill-description>|

## Standards & Templates

- Coding standards: `~/.ai/standards/<tech-stack>.md` — read when working on a specific stack.
- Templates: `~/.ai/templates/` — use when generating new files (AGENTS.md, README, adapter, standar).

## Memory

No agent here has automatic cross-session memory. `~/.ai/memory/` is for manual reference files — read on request only, no auto-injection.
