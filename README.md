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
├── standards/          # per-stack coding standards (index + per-topic files)
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

# 3. Copy skill files (not directories) from this repo
cp skills/report-by-git-changes/report-by-git-changes.md "$AI_HOME/skills/"
cp skills/venturo-customize/grademe.md "$AI_HOME/skills/"
cp skills/venturo-customize/submit-grade.md "$AI_HOME/skills/"
cp skills/subcommands/subcommands.md "$AI_HOME/skills/"

# 4. Copy adapters & templates
cp adapters/*   "$AI_HOME/adapters/"
cp templates/*  "$AI_HOME/templates/"

# 5. If using a custom $AI_HOME, replace hardcoded paths
if [ "$AI_HOME" != "$HOME/.ai" ]; then
  grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
fi

# 6. Generate global agent instructions
# Tell your agent:
# "Read ~/.ai/templates/AGENTS.global.template.md and generate ~/AGENTS.md for me."
# Re-run this step whenever you add new skills or adapters.
```

### Agent-Guided

Tell your agent:

> "Read `~/path/to/vibe-playground/templates/AGENTS.global.template.md` and set up `~/.ai/` for me."

The agent will validate that `~/.ai/` is populated, scaffold any missing structure, and generate `~/AGENTS.md`. It will ask for confirmation before overwriting any existing files.

## Skills

Skills are flat `.md` files in `~/.ai/skills/`. The Skills Index in `~/AGENTS.md` maps each skill to its trigger patterns — agents check the index first, then load the full skill file only when the trigger matches. No filesystem scan needed per request.

### Available Skills

| Skill | Trigger | Description |
|---|---|---|
| `grademe` | "grade me", "grade this session" | Grades vibe-coding from a session transcript against a 7-dimension rubric. Outputs JSON + narrative. |
| `submit-grade` | "submit grade", "upload grade" | Uploads graded session to the vibescore leaderboard. Requires API credentials. |
| `report-by-git-changes` | "report changes", "changelog" | Generates changelog-style reports from git diff or a specific commit. |
| `subcommands` | "sync skills", "sync subcommands", "/subcommands" | Syncs skills to native custom commands via pointer files. |

## Adapters

Adapters tell agents about their own runtime: which file tool to use, where sessions are stored, and what native limitations exist. They do **not** contain a skills list — that lives in `~/AGENTS.md`.

### Example Adapters

> **Note:** The adapters in `adapters/` are the author's personal examples — they show what a
> finished adapter looks like for a specific machine. Other users should generate their own via
> `~/.ai/templates/adapter.template.md`, not copy these directly.

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

### Auto-Injection (Optional)

By default, `~/AGENTS.md` is not loaded unless the agent supports a startup file or you mention
it manually. To make the agent aware of `~/.ai/` every session without explicit prompting, you
can add a steering or startup file — the exact mechanism depends on your agent.

**Delegate to your agent:**

> "Check if you support a startup or steering file that's loaded automatically every session.
> If yes, tell me what it is and what the impact would be, then ask for my approval before
> creating anything. The file should instruct you to read your global agent instructions file
> at session start."

**Or use this general format** if you want to create the file yourself:

```
---
inclusion: always   # or equivalent front-matter for your agent
---
Read your global agent instructions file (~/AGENTS.md or equivalent) at the start of this
session before responding to any request.
```

Save to the path your agent expects (e.g. `~/.kiro/steering/agents.md` for Kiro CLI).
Impact: the agent will load its global instructions — and the skills index — automatically every
session, even for unrelated projects. Turn off by removing or disabling the file.

**Adding skills to an existing steering file mid-development:**

If you want to add a skill reference later without regenerating `~/AGENTS.md`:

> "Update my startup/steering file to also remind you to check `~/.ai/skills/` when
> a request might match a skill. Ask me which skills to include before writing anything."

## Templates

Templates in `~/.ai/templates/` are prompt instructions for generating standard project files.

| Template | Generates | Usage |
|---|---|---|
| `AGENTS.global.template.md` | `~/AGENTS.md` | Global agent instructions — adapter table + skills index + when-to-act rules |
| `AGENTS.local.template.md` | `./AGENTS.md` | Project-specific instructions — stack, commands, gotchas |
| `README.template.md` | `./README.md` | Project README from codebase scan |
| `adapter.template.md` | `~/.ai/adapters/<agent>.md` | New agent adapter |
| `standard.template.md` | `~/.ai/standards/<stack>/index.md` + `~/.ai/standards/<stack>/parts/**` | Per-stack coding standards — split into index + per-topic section files, loaded on trigger match |
 
 > **Note:** All templates hardcode `~/.ai/`. See [$AI_HOME & Custom Path](#ai_home--custom-path) if you rename the directory.
 
 ## Draft Standards

The `standards/*/` folders are **personal drafts** — incomplete, opinionated templates per stack. Not ready for general use; treat them as scratch space.

## Known Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| `~/AGENTS.md` not auto-injected on all agents | Agent has no awareness of `~/.ai/` | Verify your agent supports startup files; mention it explicitly if needed |
| No cross-session memory | Every session starts fresh | Use `~/.ai/memory/` for context that must persist |
| Skills Index can go stale | Agent won't know about new skills | Regenerate `~/AGENTS.md` after adding any skill or adapter |
| Standards not auto-generated | Agent may work without stack conventions | Agent offers to generate when stack is detected — generates split index + per-topic files; requires global instructions file to be read at session start; falls back to vibe-playground clone or user-provided context if template is unavailable |

## Troubleshooting

> File paths and filenames below reflect the default setup (`~/AGENTS.md`, `~/.ai/`). Adapt to
> your actual paths if you've customized them.

**`~/AGENTS.md` generated with empty adapter table or skills index**

Cause: `~/.ai/adapters/` or `~/.ai/skills/` was empty when the agent ran the template.

Fix: populate `~/.ai/` first, using the [Installation](#installation) steps.

---

**Agent does not pick up skills or adapters**

Cause: Global agent instructions file either does not exist or is not being read at session start.

Fix:
1. Verify `~/AGENTS.md` exists — if not, generate it (see Installation step 6).
2. Check that your agent supports a startup/context file that loads automatically. The filename
   varies per agent — ask your agent: *"Do you support a startup or steering file?"*
3. If not supported natively, add your global instructions file to context manually at the start
   of each session.

---

**New skill not appearing in agent responses**

Cause: Skills Index in `~/AGENTS.md` was not updated after the skill was added.

Fix: Regenerate `~/AGENTS.md`:
> *"Regenerate `~/AGENTS.md` from `~/.ai/templates/AGENTS.global.template.md`."*

or 
> *"Read `~/.ai/skills/*.md` then update `~/AGENTS.md` Skills Index with the new skill"*

---

**Agent cannot load `~/.ai/skills/` or a custom skills path**

Cause: The agent's file access is restricted to the current workspace directory. Paths outside
the workspace (like `~/.ai/`) require explicit permission depending on the agent.

Fix per agent:

*OpenCode* — add `external_directory` permission in `opencode.json` (global config at
`~/.config/opencode/opencode.json` or per-project at `.opencode/opencode.json`):
```json
{
  "permission": {
    "external_directory": {
      "~/.ai/**": "allow"
    }
  }
}
```
If using a custom `$AI_HOME`, replace `~/.ai/` with your actual path.

*Kiro CLI* — when the agent prompts for file access approval, approve it. To pre-allow
permanently, add `allowedPaths` to the tool settings in your Kiro config, or run `/tools`
in the chat to manage permissions interactively.

*Other agents* — approve the access request when prompted, or check your agent's
documentation for an equivalent `allowedPaths` or `external_directory` config option.