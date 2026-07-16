# Adapter — opencode

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.

## Skill Discovery

Skills are flat `.md` files in `~/.ai/skills/`. Scan with:

```bash
ls ~/.ai/skills/*.md
```

or `glob("~/.ai/skills/*.md")` via the `glob` tool.

Read each file to infer its purpose from filename and content. Do NOT hardcode the list. Load only skills relevant to the current request using `read`.

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

- **No auto-skill discovery**: flat `.md` files in `~/.ai/skills/` are not auto-loaded. Scan and `read` them manually.
- **No cross-session memory**: each session starts fresh.
- **No native PR/issue integration**: use `bash` with `gh` CLI if available.
- **No conversation branching**: sessions are linear.
