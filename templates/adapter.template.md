# Adapter Template — Instructions for the Agent

You are a CLI coding agent without a dedicated file in `~/.ai/adapters/`. Create one now:

1. Identify yourself (agent name, e.g. `kiro-cli`, `opencode`, `codebuff`).
2. Fill in the template below based on what you actually know about your own runtime —
   don't guess; check your own docs/config if unsure, or leave a placeholder `<TODO>`.
3. Save the result to `~/.ai/adapters/<agent-name>.md`.
4. Keep it lean: skip any section that doesn't apply, don't restate what you already know
   about your own tools (no tool-list tables — that's redundant with your system prompt),
   and don't pad with examples that just repeat the section above them.

---
<br>
<br>

# Adapter — \<agent-name>

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.

## Skill Discovery
- Skills are flat `.md` files in `~/.ai/skills/`.
- Scan with: `<list/glob command for this agent>`
- Read a skill's full content only when its filename/purpose matches the request.
- Do NOT hardcode a skill list — new skills need no adapter changes.

## Memory
- Does this agent persist memory across sessions natively? `<yes/no>`
- If no: `~/.ai/memory/` is for manual reference files, read on request only — no
  auto-injection.

## Session Access (omit if not applicable)
- Transcript location: `<path pattern>`
- Find the latest: `<one command>`

## Limitations
- `<native gaps specific to this agent — e.g. no auto-skill loading, no PR integration>`