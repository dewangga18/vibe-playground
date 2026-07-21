# AGENTS.global Template — Instructions for the Agent

Generate or update the global agent instructions file at `~/AGENTS.md`.

## Steps

**1. Detect existing file**
- Check if `~/AGENTS.md` already exists. First-time build or regenerate — determines whether step 4 runs.
- Do NOT ask about overwrite yet (deferred to step 4 after fresh scan).

**2. Validate `~/.ai/` is populated**
- Check `~/.ai/adapters/` and `~/.ai/skills/` exist and are not empty:
```bash
  ls ~/.ai/adapters/*.md 2>/dev/null
  ls ~/.ai/skills/*.md 2>/dev/null
```
- Either missing or empty → **stop**. Inform user:
  > "`~/.ai/` is not populated yet. Copy assets from `vibe-playground/` first:"
  > ```bash
  > cp ~/path/to/vibe-playground/adapters/* ~/.ai/adapters/
  > cp ~/path/to/vibe-playground/skills/*   ~/.ai/skills/
  > cp ~/path/to/vibe-playground/templates/* ~/.ai/templates/
  > ```
  > Then re-run this template.
- Do not proceed until both folders have content.

**3. Scan available assets**
- Adapters: `ls ~/.ai/adapters/*.md` — collect agent names from filenames.
- Skills: `ls ~/.ai/skills/*.md` — read frontmatter `name`, `description`, first heading.
- If `~/AGENTS.md` already exists (step 1) with adapter/skills tables: diff fresh scan vs recorded.
  New = appears now but not recorded. Removed = recorded but no longer found.

**4. Reconcile with existing file** *(skip if step 1 found no existing `~/AGENTS.md` — go to step 5)*
- Summarize diff: new/removed adapters, new/removed skills.
- If existing file has manually added notes outside auto-tables, preserve in merge mode.
- Present options:
  > "`~/AGENTS.md` already exists. Since last generated: `<N>` new skill(s), `<N>` new adapter(s)`<, N removed>`. What would you like to do?
  > - **Overwrite** — regenerate whole file from current scan
  > - **Merge only** — update tables, keep manually added content untouched
  > - **Keep as is** — cancel, don't touch the file"
- Wait for explicit confirmation. **Keep as is** → stop.

**5. Generate `~/AGENTS.md`**
Save using the format below, applying the mode from step 4. Fill adapter table and skills index from step 3 scan — do NOT hardcode.

**6. Wire entry-point pointer**
- Read adapter's `Native Init Behavior`: `native_entry_files.global` and `reconciliation_policy`.
  If missing, check your own startup file behavior.
- **Requires explicit approval** (same risk as `adapter.template.md` step 5). Never create silently.
- **Already exists as correct pointer** → skip.
- **Doesn't exist** → ask approval, create pointer-guarded file (or symlink to `~/AGENTS.md` if supported):
  > "I'd like to create `<entry file>` so `<agent>` auto-loads `~/AGENTS.md` every session. It'll just be a pointer. OK?"
  > "👉 Global rules: `~/AGENTS.md`. Do not add content here — run `AGENTS.global.template.md` instead."
- **Exists with real content** → `reconciliation_policy: migrate-after-run` applies. Show what will be extracted into `~/AGENTS.md`, confirm before overwriting. After approval: extract useful content, convert entry file to pointer.
- If `reconciliation_policy: not-applicable` → skip.

**7. Always-active skills (optional)**
- Zero skills → skip entirely.
- Show skills list from step 3. Ask:
  > "Apply any on every request? Pick from skill list or give custom instruction."
- Nothing selected → skip.
- Selected → they'll be listed directly in `~/AGENTS.md` under `## Always-Active Skills` (step 5).
  For each selected skill, analyze its frontmatter/description and ask:
  > "When should the agent skip `<name>`? (e.g. 'No transcripts available', 'Not a git project')"
  User can accept, edit, or leave blank (no skip condition).
  Patched in Skills Index as footnote: `` `<name>` — always-active (see section above) ``.

> **Custom path?** If `$AI_HOME` is set and differs from `~/.ai/`, run:
> ```bash
> grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
> ```

---

# Project Agent Instructions

Reusable AI assets (skills, standards, templates) live in `~/.ai/` — not inside any repo.
Do not create vendor-specific directories (`.agents/`, `.opencode/`, `.kiro/`) for reusable assets.

## When to Read This File

Read this file at the start of every session. After reading:

- **No `./AGENTS.md` in project root** → offer to run `~/.ai/templates/AGENTS.local.template.md`.
- **Stack detected but no `~/.ai/standards/<stack>/index.md`** → offer to generate:
  > "No standards found for `<stack>`. Generate from `~/.ai/templates/standard.template.md`?"
  Wait for confirmation.
  - Template exists → run it.
  - Not → check `vibe-playground/` clone. If neither → ask user for conventions, generate from that.
- **User request is ambiguous** → check Skills Index before asking.

## Tool Adapters

Identify which CLI agent you are, then read the matching adapter:

| Agent | Adapter |
|---|---|
| Freebuff (CodebuffAI) | `~/.ai/adapters/freebuff.md` |
| Kiro CLI | `~/.ai/adapters/kiro.md` |
| OpenCode | `~/.ai/adapters/opencode.md` |
<!-- FILL from ~/.ai/adapters/*.md scan — add rows for new adapters -->

Read the matching adapter **before** responding — it tells you how to read files and access sessions.

## No adapter for you yet?

Read `~/.ai/templates/adapter.template.md` and create one now.
Save to `~/.ai/adapters/<agent-name>.md`.

## Always-Active Skills

<!-- Include only if step 7 selected skills. Omit entirely if none. -->
Read every applicable skill at session start. Skip any whose `Skip if` condition matches current context. If `Skip if` is blank, always read that skill.

| Skill | File | Skip if |
|---|---|---|
| `<name>` | `~/.ai/skills/<name>.md` | `<condition or blank>` |

## Skills Index

<!-- FILL from ~/.ai/skills/*.md scan — one row per file. For skills listed in Always-Active Skills above, use footnote: "`<name>` — always-active (see section above)". -->
| Skill | Trigger pattern | Description | File |
|---|---|---|---|
| `<name>` | `<example phrases that should load this skill>` | `<one-line description>` | `~/.ai/skills/<name>.md` |

When a user request matches a trigger pattern, read the full skill file before responding.
Do not load skills preemptively — only when the trigger pattern matches.

## Standards

Coding standards live in `~/.ai/standards/<tech-stack>/index.md` — an **index**, not the full standard. It links to focused section files under `~/.ai/standards/<tech-stack>/parts/`, each gated behind a trigger pattern — same discovery as Skills Index. Read the index first, then only sections whose trigger matches.
Exception: on a brand-new/empty project (no existing source files), read every section once up front.
If no index exists, offer to generate (see "When to Read This File").

## Memory

No agent here has automatic cross-session memory. `~/.ai/memory/` is for manual reference files — read on request only, no auto-injection.
