# Adapter — kiro-cli

## File Access
- Read files with: `read` tool
- Load skill on trigger match: `~/.ai/skills/<skill-name>.md`

## Session Access
- Transcript: `~/.kiro/sessions/cli/<session-uuid>.jsonl`
- Latest: `ls -t ~/.kiro/sessions/cli/*.jsonl | head -1`
- No export command or API. JSONL is the only access path.

## Limitations
- **No PR/issue integration**: use `shell` with `gh` CLI if available.
- **No conversation branching**: sessions are linear.