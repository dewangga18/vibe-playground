# Adapter — freebuff

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.
Skills index and trigger patterns live in `~/AGENTS.md` — read that first.

## File Access
- Read files with: `read_files` tool
- Read a skill file when its name/trigger matches the current request: `~/.ai/skills/<skill-name>.md`

## Memory

Freebuff does not persist memory across sessions. `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Session Access

- Transcript location: `~/.config/manicode/projects/<project>/chats/<timestamp>/log.jsonl`
- `<project>` = basename of the working directory (e.g. `/Users/a/proj` → `proj`)
- `<timestamp>` = ISO chat creation time (e.g. `2026-07-16T01-28-35.327Z`)
- Find the latest: `ls -t ~/.config/manicode/projects/<project>/chats/ | head -1`
- Each chat folder also contains: `chat-messages.json`, `chat-meta.json`, `run-state.json`

## Limitations

- **No cross-session memory**: each session starts fresh.
- **No native skill tool**: the native `skill` tool loads from pre-installed dirs, not `~/.ai/skills/`. Load skills manually with `read_files`.
- **No arbitrary API calls**: only web fetch supported. Browser automation (`browser-use`) requires Chrome.
