# Adapter — opencode

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.
Skills index and trigger patterns live in `~/AGENTS.md` — read that first.

## File Access
- Read files with: `read` tool, or `glob("~/.ai/skills/*.md")` via the `glob` tool
- Read a skill file when its name/trigger matches the current request: `~/.ai/skills/<skill-name>.md`

## Memory

opencode does not persist memory across sessions. `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Session Access

Session data is stored in SQLite at `~/.local/share/opencode/opencode.db`.

Find the latest session:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "SELECT id FROM session ORDER BY time_created DESC LIMIT 1;"
```

Export a transcript:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "SELECT json_object(
    'role', json_extract(m.data, '$.role'),
    'timestamp', m.time_created,
    'model', json_extract(m.data, '$.model.modelID'),
    'parts', (SELECT json_group_array(json_object(
      'type', json_extract(p.data, '$.type'),
      'text', json_extract(p.data, '$.text'),
      'tool', json_extract(p.data, '$.tool')
    )) FROM part p WHERE p.message_id = m.id)
  ) FROM message m
  WHERE m.session_id = '{SESSION_ID}'
  ORDER BY m.time_created ASC;"
```

No API, export command, or internal memory hook. SQLite is the only access path.

## Limitations

- **No cross-session memory**: each session starts fresh.
- **No native PR/issue integration**: use `bash` with `gh` CLI if available.
- **No conversation branching**: sessions are linear.
