# Adapter — kiro-cli

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.
Skills index and trigger patterns live in `~/AGENTS.md` — read that first.

## File Access
- Read files with: `read` tool (built-in)
- Read a skill file when its name/trigger matches the current request: `~/.ai/skills/<skill-name>.md`

## Memory

Kiro CLI does not persist memory across sessions. `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Session Access

- Transcript location: `~/.kiro/sessions/cli/<session-uuid>.jsonl`
- Find the latest: `ls -t ~/.kiro/sessions/cli/*.jsonl | head -1`
- No export command or API. JSONL is the only access path.

## Limitations

- **No cross-session memory**: each session starts fresh.
- **No PR/issue integration**: use `shell` with `gh` CLI if available.
- **No conversation branching**: sessions are linear.
