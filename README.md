# vibe-playground

A portable AI agent toolkit — reusable skills, adapter configs, and prompt templates that live outside any project repo.

## About

`vibe-playground` is the draft workspace for building and maintaining assets that power AI coding agents across all your projects. The actual assets get deployed to `~/.ai/` where any supported agent can discover and use them at runtime.

The agents themselves are never modified. Instead, you teach them how to find and use shared skills, memory, and standards through a global `~/AGENTS.md` and per-agent adapter files.

## How It Works

There are two directories involved:

```
vibe-playground/        ← you edit here (source/draft)
    ↓  deploy
~/.ai/                  ← agents read from here (runtime)
├── adapters/           # per-agent runtime quirks: file tools, session paths, limitations
├── memory/             # manual reference files (read on request, no auto-injection)
├── skills/             # reusable task prompts — flat .md files
├── standards/          # per-stack coding standards
└── templates/          # templates for generating AGENTS.md, README, adapters, standards
```

The full agent flow per session:

```
1. Agent starts → reads ~/AGENTS.md (entry point)
2. ~/AGENTS.md  → agent identifies itself → reads matching ~/.ai/adapters/<agent>.md
3. adapter      → agent knows: which file tool to use, where sessions live, limitations
4. User request → agent checks Skills Index in ~/AGENTS.md → loads skill file if trigger matches
5. New project  → no ./AGENTS.md found → agent offers to generate one from AGENTS.local.template.md
```

Nothing is auto-loaded — agents decide based on trigger patterns in the Skills Index.

## `~/AGENTS.md` — The Entry Point

`~/AGENTS.md` is the file that makes the entire system work. It is:

- **Generated** from `~/.ai/templates/AGENTS.global.template.md` — not hand-written
- **Read by agents** at the start of every session (requires startup file support per agent)
- **Contains:** adapter table, skills index with trigger patterns, and when-to-act rules

Without `~/AGENTS.md`, agents have no awareness of `~/.ai/` at all.

Re-generate it whenever you add new skills or adapters:

> Tell your agent: *"Read `~/.ai/templates/AGENTS.global.template.md` and regenerate `~/AGENTS.md`."*

## `$AI_HOME` & Custom Path

All files in this repo hardcode `~/.ai/` as the default toolkit path. If you want a different location:

**1. Set the env var in your shell profile:**
```bash
export AI_HOME=~/.myai   # or any path you prefer
```

**2. After deploying, replace all hardcoded paths in `$AI_HOME`:**
```bash
grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
```

The agent-guided install (see below) handles step 2 automatically if `$AI_HOME` is set.

## Installation

### Manual

```bash
# 1. Set your preferred path (skip to use the default ~/.ai/)
export AI_HOME=~/.ai

# 2. Create the directory structure
mkdir -p "$AI_HOME"/{adapters,memory,skills,standards,templates}

# 3. Copy assets from this repo
cp adapters/*   "$AI_HOME/adapters/"
cp skills/*     "$AI_HOME/skills/"
cp templates/*  "$AI_HOME/templates/"

# 4. If using a custom $AI_HOME, replace hardcoded paths
if [ "$AI_HOME" != "$HOME/.ai" ]; then
  grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
fi

# 5. Generate global agent instructions
# Tell your agent:
# "Read ~/.ai/templates/AGENTS.global.template.md and generate ~/AGENTS.md for me."
# Re-run this step whenever you add new skills or adapters.
```

### Agent-Guided

Tell your agent:

> "Read `~/Documents/vibe-playground/templates/AGENTS.global.template.md` and set up `~/.ai/` for me."

The agent will validate that `~/.ai/` is populated, scaffold any missing structure, and generate `~/AGENTS.md`. It will ask for confirmation before overwriting any existing files.

## Skills

Skills are flat `.md` files in `~/.ai/skills/`. The Skills Index in `~/AGENTS.md` maps each skill to its trigger patterns — agents check the index first, then load the full skill file only when the trigger matches. No filesystem scan needed per request.

### Available Skills

| Skill | Trigger | Description |
|---|---|---|
| `grademe` | "grade me", "grade this session" | Grades the user's vibe-coding practice from a session transcript against a 7-dimension rubric. Outputs JSON + narrative. |
| `report-by-git-changes` | "report changes", "changelog" | Generates a concise changelog-style report from current git diff or a specific commit. |

### Adding a Skill

1. Create `~/.ai/skills/<skill-name>.md` with a frontmatter block:

```markdown
---
name: my-skill
description: What this skill does in one sentence.
---

# My Skill

Instructions for the agent...
```

2. Regenerate `~/AGENTS.md` so the new skill appears in the Skills Index:
   > *"Regenerate `~/AGENTS.md` from `~/.ai/templates/AGENTS.global.template.md`."*

No adapter changes needed.

## Adapters

Adapters tell agents about their own runtime: which file tool to use, where sessions are stored, and what native limitations exist. They do **not** contain a skills list — that lives in `~/AGENTS.md`.

### Supported Agents

| Agent | Adapter | Notes |
|---|---|---|
| Kiro CLI | `adapters/kiro.md` | Sessions in `~/.kiro/sessions/cli/*.jsonl` |
| Freebuff (CodebuffAI) | `adapters/freebuff.md` | Sessions in `~/.config/manicode/projects/` |
| OpenCode | `adapters/opencode.md` | Sessions in SQLite at `~/.local/share/opencode/opencode.db` |

### Adding an Adapter for a New Agent

Tell your agent:

> "Read `~/.ai/templates/adapter.template.md` and create an adapter for yourself."

The agent fills in its own runtime details and saves to `~/.ai/adapters/<agent-name>.md`.
Then regenerate `~/AGENTS.md` so the new adapter appears in the adapter table.

## Templates

Templates in `~/.ai/templates/` are prompt instructions for generating standard project files.

| Template | Generates | Usage |
|---|---|---|
| `AGENTS.global.template.md` | `~/AGENTS.md` | Global agent instructions — adapter table + skills index + when-to-act rules |
| `AGENTS.local.template.md` | `./AGENTS.md` | Project-specific instructions — stack, commands, gotchas |
| `README.template.md` | `./README.md` | Project README from codebase scan |
| `adapter.template.md` | `~/.ai/adapters/<agent>.md` | New agent adapter |
| `standar.template.md` | `~/.ai/standards/<stack>.md` | Per-stack coding standards |

> **Note:** All templates hardcode `~/.ai/`. See [$AI_HOME & Custom Path](#ai_home--custom-path) if you rename the directory.

## Known Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| `~/AGENTS.md` not auto-injected on all agents | Agent has no awareness of `~/.ai/` | Verify your agent supports startup files; mention it explicitly if needed |
| No cross-session memory | Every session starts fresh | Use `~/.ai/memory/` for context that must persist |
| Skills Index can go stale | Agent won't know about new skills | Regenerate `~/AGENTS.md` after adding any skill or adapter |
| Standards not auto-generated | Agent may work without stack conventions | Agent will prompt user to generate when stack is detected but no standards file exists |

## Troubleshooting

**`~/AGENTS.md` generated with empty adapter table or skills index**

Cause: `~/.ai/adapters/` or `~/.ai/skills/` was empty when the agent ran the template.

Fix: populate `~/.ai/` first, then regenerate:
```bash
cp ~/Documents/vibe-playground/adapters/* ~/.ai/adapters/
cp ~/Documents/vibe-playground/skills/*   ~/.ai/skills/
cp ~/Documents/vibe-playground/templates/* ~/.ai/templates/
```
Then ask the agent to regenerate `~/AGENTS.md`.

---

**Agent does not pick up skills or adapters**

Cause: `~/AGENTS.md` either does not exist or is not being read at session start.

Fix:
1. Verify `~/AGENTS.md` exists — if not, generate it (see Installation step 5).
2. Check that your agent supports a startup/context file (`AGENTS.md`, `CLAUDE.md`, etc.).
3. If not supported natively, add `~/AGENTS.md` to your agent's context manually at the start of each session.

---

**New skill not appearing in agent responses**

Cause: Skills Index in `~/AGENTS.md` was not updated after the skill was added.

Fix: Regenerate `~/AGENTS.md`:
> *"Regenerate `~/AGENTS.md` from `~/.ai/templates/AGENTS.global.template.md`."*

## Contributing / Adding Assets

This repo is intentionally minimal — each file should earn its place.

- **New skill**: add `skills/<name>.md`, follow the frontmatter convention, then regenerate `~/AGENTS.md`
- **New adapter**: add `adapters/<agent>.md`, follow `templates/adapter.template.md`, then regenerate `~/AGENTS.md`
- **New standard**: add `standards/<stack>.md`, follow `templates/standar.template.md`
- **New template**: add `templates/<name>.template.md`, document it in this README

No build step, no registry, no framework — just flat `.md` files.
