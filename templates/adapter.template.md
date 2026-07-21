# Adapter Template — Instructions for the Agent

You are a CLI coding agent without a dedicated file in `~/.ai/adapters/`. Create one now:

1. Identify yourself (agent name, e.g. `kiro-cli`, `opencode`, `codebuff`).
2. Fill in the template below based on what you actually know about your own runtime —
   don't guess; check your own docs/config if unsure, or leave a placeholder `<TODO>`.
   For the **Native Init Behavior** block specifically,
   verify against your own docs before writing "none" — it affects how `AGENTS.global.template.md`
   reconciles with your native tooling.
3. Save the result to `~/.ai/adapters/<agent-name>.md`.
4. Keep it lean: skip any section that doesn't apply, don't restate what you already know
   about your own tools (no tool-list tables — that's redundant with your system prompt),
   and don't pad with examples that just repeat the section above them.
5. After saving, ask about auto-injection:
   - Check if you support a startup file loaded every session (your `native_entry_files.global`).
   - If yes, present two options:
     - **Auto-inject:** Create startup file pointing to your global instructions. State the file, its impact on all sessions, and ask explicit approval(e.g. `~/AGENTS.md` or your agent's global instructions file).
     - **Manual:** No file. User mentions global instructions when needed.
   - Don't name specific filenames — user approves based on impact.
   - No startup file support? Inform user, suggest manual mention each session.

> **Note:** Skills index (what skills exist and their triggers) lives in your global agent instructions file —
> not in this adapter. This file only covers runtime quirks specific to this agent.

> **Custom path?** If `$AI_HOME` is set and differs from `~/.ai/`, run:
> ```bash
> grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
> ```

---


# Adapter — \<agent-name>

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.
Skills index and trigger patterns live in your global agent instructions file — read that first.

## File Access
- Read files with: `<tool name for this agent, e.g. read, read_files, cat via bash>`
- Read a skill file when its name/trigger matches the current request: `~/.ai/skills/<skill-name>.md`

## Memory
- Does this agent persist memory across sessions natively? `<yes/no>`
- If no: `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Native Init Behavior
<!-- Does this agent have a built-in subcommand that scans a codebase and generates a
     project-level instructions file (like Claude Code's `/init` → CLAUDE.md)? Check your
     own docs — don't guess. If none, write "none" for native_init_command and skip the rest. -->
- native_init_command: `<e.g. /init, or: none>`
- native_entry_files:
  - global: `<e.g. ~/.claude/CLAUDE.md, or: n/a>`
  - project: `<e.g. ./CLAUDE.md, or: n/a>`
  - subdirectory: `<e.g. ./<dir>/CLAUDE.md, or: n/a>`
- reconciliation_policy: `<pointer-guarded | migrate-after-run | not-applicable>`
  <!-- pointer-guarded: entry file = redirect to AGENTS.md, guarded against silent overwrite.
       migrate-after-run: native_init_command content → extract to AGENTS.md, convert entry to pointer.
       not-applicable: no native_init_command or agent doesn't load entry files this way. -->
- notes: `<e.g. "loads all hierarchy levels simultaneously, more specific wins">`

## Session Access *(omit if not applicable)*
- Transcript location: `<path pattern>`
- Find the latest: `<one command>`

## Limitations
`<native gaps specific to this agent — e.g. no PR integration, no browser automation>`