# Subcommands Sync

Meta-skill that syncs skills from `~/.ai/skills/` to native custom commands in Kiro, OpenCode, Freebuff, or other CLI agents. **Pointer-only** — never duplicates skill content.

## How It Works

A 6-step workflow:

1. **Discover** — scan skills from `~/.ai/skills/*.md` (or custom path / GitHub fetch / pasted markdown)
2. **Detect agent** — detect which agent you're in via cwd markers (`.kiro/`, `.opencode/`, `.agents/`)
3. **Scan integrated** — check which skills are already registered as native commands
4. **Diff** — subtract already-integrated skills, show what's pending
5. **Generate** — create pointer files or symlinks (only after user confirms)
6. **Report** — summarize results per agent

## Params

```
-path=<dir>       Scan a custom skills directory instead of ~/.ai/skills/
-github=<url>     Fetch a single skill file from a GitHub raw URL
-md=<markdown>    Use pasted markdown content as the skill source
```

All optional. Default: scan `~/.ai/skills/`. Multiple params can be combined.

## Getting Started

Before `subcommands` can sync other skills, you need to install `subcommands.md` itself as a custom command.

### Freebuff

```
Install ~/.ai/skills/subcommands.md as a /skill:subcommands command
```

The agent will create a global symlink at `~/.agents/skills/subcommands/SKILL.md` → `~/.ai/skills/subcommands.md`, making it available in every repo.

### Kiro

```
Install ~/.ai/skills/subcommands.md as a custom /subcommands command
```

The agent will create `~/.kiro/prompts/subcommands.md` with a pointer to the skill file.

### OpenCode

```
Register ~/.ai/skills/subcommands.md as a /subcommands command
```

The agent will create `~/.opencode/commands/subcommands.md` with a pointer to the skill file.

### Other agents

```
Read ~/.ai/skills/subcommands.md and follow its instructions to register it as a custom command
```

The agent will inspect its own custom command mechanism and create the appropriate pointer based on the skill's instructions.

---
> **You should restart your terminal session after installing `subcommands` as a native command. <br>**

Once `subcommands` is installed as a native command (`/subcommands` for Kiro/OpenCode, `/skill:subcommands` for Freebuff), you can run it to sync all other skills:

```
/subcommands
# or for Freebuff:
/skill:subcommands
```

## Output per Agent

| Agent | Pointer location | Invocation | Scope |
|---|---|---|---|
| **Kiro** | `~/.kiro/prompts/<id>.md` | `/<id>` | Global |
| **OpenCode** | `~/.opencode/commands/<id>.md` | `/<id>` | Global |
| **Freebuff** | `~/.agents/skills/<id>/SKILL.md` (global only) | `/skill:<id>` | Global |

## Freebuff: Global Only

Freebuff skills are installed globally at `~/.agents/skills/` — available in every repo. No per-repo configuration needed.

## Usage Examples

```
# Default — scan ~/.ai/skills/ and sync all unregistered skills
/subcommands

# Sync from a custom directory
/subcommands -path=./custom-skills

# Fetch a skill from GitHub first, then sync the rest
/subcommands -github=https://raw.githubusercontent.com/user/repo/main/skills/grademe.md

# Sync from pasted markdown content
/subcommands -md="---\nname: my-skill\n---\n# My Skill\n..."
```
