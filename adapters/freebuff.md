# Adapter — freebuff

## File Access
- Read files with: `read_files` tool
- Load skill on trigger match: `~/.ai/skills/<skill-name>.md`

## Session Access
- Transcript: `~/.config/manicode/projects/<project>/chats/<timestamp>/log.jsonl`
- `<project>` = basename of working directory
- `<timestamp>` = ISO chat creation time (e.g. `2026-07-16T01-28-35.327Z`)
- Latest: `ls -t ~/.config/manicode/projects/<project>/chats/ | head -1`
- Each chat folder also contains: `chat-messages.json`, `chat-meta.json`, `run-state.json`

## Limitations
- **No native skill tool**: the `skill` tool loads from pre-installed dirs, not `~/.ai/skills/`. Load manually with `read_files`.
- **No arbitrary API calls**: only web fetch supported. Browser automation requires Chrome.