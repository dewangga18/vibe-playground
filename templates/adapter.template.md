# Adapter Template — Instructions for the Agent

You are a CLI coding agent without a dedicated file in `~/.ai/adapters/`. Create one now:

1. Identify yourself (agent name, e.g. `kiro-cli`, `opencode`, `codebuff`).
2. Fill in the template below based on what you actually know about your own runtime —
   don't guess; check your own docs/config if unsure, or leave a placeholder `<TODO>`.
3. Save the result to `~/.ai/adapters/<agent-name>.md`.
4. Keep it lean: skip any section that doesn't apply, don't restate what you already know
   about your own tools (no tool-list tables — that's redundant with your system prompt),
   and don't pad with examples that just repeat the section above them.

> **Note:** Skills index (what skills exist and their triggers) lives in `~/AGENTS.md` —
> not in this adapter. This file only covers runtime quirks specific to this agent.

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

# Adapter — \<agent-name>

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.
Skills index and trigger patterns live in `~/AGENTS.md` — read that first.

## File Access
- Read files with: `<tool name for this agent, e.g. read, read_files, cat via bash>`
- Read a skill file when its name/trigger matches the current request: `~/.ai/skills/<skill-name>.md`

## Memory
- Does this agent persist memory across sessions natively? `<yes/no>`
- If no: `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Session Access *(omit if not applicable)*
- Transcript location: `<path pattern>`
- Find the latest: `<one command>`

## Limitations
`<native gaps specific to this agent — e.g. no PR integration, no browser automation>`
