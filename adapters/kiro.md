# Adapter — kiro-cli

`~/.ai/` is the single source of truth for reusable skills, memory, and docs.

## Skill Discovery

Skills are flat `.md` files in `~/.ai/skills/`. Scan with:

```bash
ls ~/.ai/skills/*.md
```

Read each file to infer its purpose from filename and content. Do NOT hardcode the list. Load only skills relevant to the current request.

## Memory

Kiro CLI does not persist memory across sessions. `~/.ai/memory/` is for manual reference files, read on request only — no auto-injection.

## Session Access

- Transcript location: `~/.kiro/sessions/cli/<session-uuid>.jsonl`
- Find the latest: `ls -t ~/.kiro/sessions/cli/*.jsonl | head -1`
- No export command or API. JSONL is the only access path.

## Limitations

- **No cross-session memory**: each session starts fresh.
- **No native skill tool**: load skills by reading `~/.ai/skills/<skill-name>.md` directly.
- **No auto-skill loading**: the agent reads skill descriptions and decides — no keyword-triggered auto-load.
- **No PR/issue integration**: use `shell` with `gh` CLI if available.
- **No conversation branching**: sessions are linear.
