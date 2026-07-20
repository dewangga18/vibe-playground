# Adapter Template — Instructions for the Agent

You are a CLI coding agent without a dedicated file in `~/.ai/adapters/`. Create one now:

1. Identify yourself (agent name, e.g. `kiro-cli`, `opencode`, `codebuff`).
2. Fill in the template below based on what you actually know about your own runtime —
   don't guess; check your own docs/config if unsure, or leave a placeholder `<TODO>`.
   For the **Native Init Behavior** and **Native Always-Active Mechanism** blocks specifically,
   verify against your own docs before writing "none" — these affect how other templates
   (`AGENTS.local.template.md`, `AGENTS.global.template.md`) reconcile with your native tooling.
3. Save the result to `~/.ai/adapters/<agent-name>.md`.
4. Keep it lean: skip any section that doesn't apply, don't restate what you already know
   about your own tools (no tool-list tables — that's redundant with your system prompt),
   and don't pad with examples that just repeat the section above them.
5. After saving the adapter, ask the user about auto-injection:
   - Check if you support a startup or steering file that loads automatically every session
     (this is your `native_entry_files.global` from the block below, if you filled it in).
   - If yes, present **two options** to the user:
     - **Option A — auto-inject:** Create a startup/steering file that instructs you to read
       your global agent instructions file (e.g. `~/AGENTS.md` or equivalent for your agent) at 
       the start of every session. State what the file is, where it lives, and that it will affect 
       all sessions including unrelated projects. Ask for explicit approval before creating anything.
     - **Option B — manual:** No file created. User mentions `~/<agent-name>.md` when needed.
   - Do not name specific filenames (e.g. `AGENTS.md`, `CLAUDE.md`) in the options — you know
     what your own startup file is called. Let the user approve based on impact, not file names.
   - If you don't support a startup file, inform the user and suggest mentioning `~/<agent-name>.md`
     manually at the start of each session.

> **Note:** Skills index (what skills exist and their triggers) lives in your global agent instructions file —
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
  <!-- pointer-guarded: entry file should be kept as a redirect to AGENTS.md, guarded against
       being silently overwritten by native_init_command.
       migrate-after-run: if native_init_command writes real content, that content should be
       extracted into AGENTS.md and the entry file converted back to a pointer.
       not-applicable: no native_init_command, or this agent doesn't load entry files this way. -->
- notes: `<e.g. "loads all hierarchy levels simultaneously, more specific wins">`

## Native Always-Active Mechanism
<!-- Separate from Native Init Behavior above. This is about RULES the agent applies on every
     request regardless of trigger match — e.g. hooks, steering docs, or an interview-based
     setup flow. Does this agent support that outside of just reading an instructions file? -->
- supported: `<yes/no>`
- mechanism: `<e.g. hooks config at .agent/hooks.json, steering docs at .agent/steering/*.md, or: none>`
- format: `<brief note on expected format/schema, or: n/a>`
- If `supported: no` → always-active rules for this agent fall back to `~/ALWAYS.md`
  (read via the pointer set up in step 5 above).

## Session Access *(omit if not applicable)*
- Transcript location: `<path pattern>`
- Find the latest: `<one command>`

## Limitations
`<native gaps specific to this agent — e.g. no PR integration, no browser automation>`