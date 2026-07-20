# AGENTS.global Template — Instructions for the Agent

Generate or update the global agent instructions file at `~/AGENTS.md`.

## Steps

**1. Detect existing file**
- Check if `~/AGENTS.md` already exists. Note this as first-time build or regenerate — it
  determines whether step 4 (reconcile) runs.
- Do NOT ask about overwrite yet. That decision happens in step 4, once fresh scan results
  actually exist to diff against.

**2. Validate `~/.ai/` is populated**
- Check that `~/.ai/adapters/` and `~/.ai/skills/` exist and are not empty:
```bash
  ls ~/.ai/adapters/*.md 2>/dev/null
  ls ~/.ai/skills/*.md 2>/dev/null
```
- If either folder is missing or empty → **stop**. Inform the user:
  > "`~/.ai/` is not populated yet. Copy assets from `vibe-playground/` first:"
  > ```bash
  > cp ~/path/to/vibe-playground/adapters/* ~/.ai/adapters/
  > cp ~/path/to/vibe-playground/skills/*   ~/.ai/skills/
  > cp ~/path/to/vibe-playground/templates/* ~/.ai/templates/
  > ```
  > Then ask the agent to run this template again.
- Do not proceed until both folders have content.

**3. Scan available assets**
- Adapters: `ls ~/.ai/adapters/*.md` — collect agent names from filenames.
- Skills: `ls ~/.ai/skills/*.md` — read frontmatter `name`, `description`, first heading.
- Identify the currently active adapter (`~/.ai/adapters/<agent-name>.md`). Read its
  `Native Always-Active Mechanism` block (`supported`, `mechanism`, `format`).
  - If `supported: no`, or the adapter doesn't declare this block, or no adapter exists yet →
    fall back to `~/ALWAYS.md`.
- If `~/AGENTS.md` already exists (from step 1) and has `## Tool Adapters` / `## Skills Index`
  tables, diff the freshly scanned adapters and skills against what's already recorded.
  For skills specifically: before diffing, also read the always-active source identified
  above (`~/ALWAYS.md`, or the native mechanism's file if `supported: yes`), if it exists, and
  treat any skill listed there as already recorded too — it's intentionally omitted from the
  Skills Index table (see step 5), not missing. Union both sources, then diff.
  Items that appear now but weren't recorded in either source = **new**. Items recorded but
  no longer found = **removed**.

**4. Reconcile with existing file** *(skip entirely if step 1 found no existing `~/AGENTS.md`
— go straight to step 5 with a fresh build)*
- Summarize the diff from step 3: new/removed adapters, new/removed skills.
- If the existing file contains any content outside the auto-filled tables and standard
  boilerplate (e.g. manually added notes), treat that as curated and preserve it in merge mode.
- Present the user with three options:
  > "`~/AGENTS.md` already exists. Since last generated: `<N>` new skill(s), `<N>` new
  > adapter(s)`<, N removed>`. What would you like to do?
  > - **Overwrite** — regenerate the whole file from the current scan
  > - **Merge only** — update the Tool Adapters / Skills Index tables, keep any manually
  >   added content elsewhere in the file untouched
  > - **Keep as is** — cancel, don't touch the file"
- Wait for explicit confirmation before continuing.
  - **Keep as is** → stop here. Do not modify `~/AGENTS.md`.

**5. Generate `~/AGENTS.md`**

Save using the format below, applying the mode chosen in step 4 (fresh build, full overwrite,
or targeted merge). Fill the adapter table and skills index from the scan in step 3 — do NOT hardcode.

**6. Wire entry-point pointer**
- Read the active adapter's `Native Init Behavior` block: `native_entry_files.global` and
  `reconciliation_policy`. If the adapter doesn't declare this block, fall back to: check if
  your agent reads a different startup file by default, using best knowledge of your own runtime.
- This step creates or modifies a file that auto-loads every session, including unrelated
  projects — the same category of action `adapter.template.md` step 5 requires explicit
  approval for. Never create or overwrite it silently; always confirm first.
- **Entry file already exists as a correct pointer** → nothing to do, skip silently.
- **Entry file doesn't exist yet** → ask for approval, then create it as a pointer-guarded
  file (or symlink to `~/AGENTS.md` if supported):
  > "I'd like to create `<entry file>` so `<agent>` auto-loads `~/AGENTS.md` every session.
  > It'll just be a pointer — no content lives there directly. OK to create it?"
  > "👉 Global rules: `~/AGENTS.md`. Do not add content directly here — run
  > `AGENTS.global.template.md` instead."
- **Entry file exists with real content** (not just a pointer — e.g. from native init or hand
  edits) → this means `reconciliation_policy: migrate-after-run` applies. Show the user what
  will be extracted into `~/AGENTS.md` and ask for confirmation before overwriting the entry
  file — it holds existing, possibly hand-written content that this step would replace. Only
  after approval: extract anything useful into `~/AGENTS.md` (e.g. as a manual note under the
  relevant section), then convert the entry file to the pointer-guarded content above.
- If `reconciliation_policy: not-applicable` → skip this step.

**7. Always-active rules (optional)**

- If step 3 returns zero skills AND this is a first-time build → skip this step entirely.

*First-time build* (no prior `~/AGENTS.md`):
- Show the full list of skills discovered in step 3.
- Ask:
  > "Are there any rules/instructions the agent should apply on EVERY request — not just
  > when a trigger matches? You can pick from the skills above, or give a custom instruction."
- Wait for the user's input. If nothing is selected/provided → skip, don't create any file.

*Regenerate with new skills* (`~/AGENTS.md` already exists, step 3 found new skills):
- Do NOT re-ask about skills that were already recorded — "recorded" means listed in the
  Skills Index table OR already present in the always-active source (`~/ALWAYS.md` or native
  mechanism), per the union check in step 3. A skill promoted to always-active in a prior run
  must never resurface here as "new".
- For each genuinely new skill, the agent analyzes its frontmatter/description/trigger: does
  it look suitable for always-on application (broad, context-independent) vs.
  narrow/trigger-based?
- If at least one looks suitable → ask the user, showing ONLY those candidates:
  > "The new skill `<name>` looks like it should be applied on every request (because
  > `<reason>`). Add it to the always-active rules?"
- If there are no new skills, or none look suitable → skip silently, don't ask the user.

**Where to write it:**
- If the active adapter's `Native Always-Active Mechanism` has `supported: yes` → write/append
  to the path declared in `mechanism`, following the schema noted in `format`.
- Otherwise → write/append to `~/ALWAYS.md`. If that file already exists, show its contents
  and confirm append vs. replace first (mirror the pattern used for `~/AGENTS.md` in step 4).

**8. Wire the always-active pointer into `~/AGENTS.md`**
- Only relevant when step 7's target was the generic `~/ALWAYS.md` fallback. If the rules were
  routed to a native mechanism (`supported: yes`), do NOT add a "read now" instruction — just
  an informational note, since the native mechanism is already auto-loaded by the tool itself,
  outside the AGENTS.md read path.
- Insert point: after `## No adapter for you yet?`, before `## Skills Index` — for both first
  build and later patching.
- Also patch the Skills Index table generated in step 5: for every skill just added to the
  always-active source in this step, remove its full row and replace it with the footnote
  line described in the Skills Index template (`` `<name>` — always-active, see ~/ALWAYS.md ``,
  or the native mechanism's file). Step 5 runs before step 7's decisions exist, so this table
  needs the same-run patch to stay consistent.

Generic fallback:<br>
```bash
## Always-Active Skills

If `~/ALWAYS.md` exists, read it now and load every skill listed in it before responding
to any request this session.
```

Native fallback (informational only, not an executable instruction):<br>
```bash
## Always-Active Rules
This agent uses <AgentName>'s native always-active mechanism for these rules — see
`~/.ai/adapters/<agent-name>.md` (Native Always-Active Mechanism block) for location and
format. Not routed through this file.
```

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

# Project Agent Instructions

Reusable AI assets (skills, standards, templates) live in `~/.ai/` — not inside any repo.
Do not create vendor-specific directories (`.agents/`, `.opencode/`, `.kiro/`) for reusable assets.

## When to Read This File

Read this file at the start of every session. After reading:

- **No `./AGENTS.md` in project root** → generate one before starting work:
  tell the user and offer to run `~/.ai/templates/AGENTS.local.template.md`.
- **Stack detected but no `~/.ai/standards/<stack>.md`** → offer to create one:
  > "No standards found for `<stack>`. Generate from `~/.ai/templates/standard.template.md`?"
  Wait for user confirmation before proceeding.
  - If `~/.ai/templates/standard.template.md` exists → run it to generate the file.
  - If not → check if a `vibe-playground/` clone is available and offer to copy the template from there.
  - If neither → ask the user to provide stack conventions or rules to follow, then generate
    `~/.ai/standards/<stack>.md` from that context directly.
- **User request is ambiguous** → check the Skills Index below before asking for clarification.
  A matching skill likely covers it.

## Tool Adapters

Identify which CLI agent you are, then read the matching adapter file:

| Agent | Adapter |
|---|---|
| Freebuff (CodebuffAI) | `~/.ai/adapters/freebuff.md` |
| Kiro CLI | `~/.ai/adapters/kiro.md` |
| OpenCode | `~/.ai/adapters/opencode.md` |
<!-- FILL from ~/.ai/adapters/*.md scan — add rows for any new adapters -->

Read the matching adapter **before** responding — it tells you how to read files and access sessions.

## No adapter for you yet?

Read `~/.ai/templates/adapter.template.md` and create one now.
Save to `~/.ai/adapters/<agent-name>.md`.

## Always-Active Skills

<!-- Include this section ONLY if ~/ALWAYS.md was actually created in step 7/8.
     Omit it entirely if no always-active skills were selected or if step 3 found zero skills. -->
If `~/ALWAYS.md` exists, read it now and load every skill listed in it before responding
to any request this session.

## Skills Index

<!-- FILL from ~/.ai/skills/*.md scan — one row per file, EXCEPT skills already listed in
     ~/ALWAYS.md's Always-Active Skills table (or the native always-active mechanism, if
     supported: yes). Those already load every session — a trigger-based row for them is
     redundant and misleading. For those, add one footnote line instead of a full row:
     "`<name>` — always-active, see ~/ALWAYS.md" (or the native mechanism's file). -->
| Skill | Trigger pattern | Description | File |
|---|---|---|---|
| `<name>` | `<example phrases that should load this skill>` | `<one-line description>` | `~/.ai/skills/<name>.md` |

When a user request matches a trigger pattern, read the full skill file before responding.
Do not load skills preemptively — only when the trigger pattern matches.

## Standards

Coding standards live in `~/.ai/standards/<tech-stack>.md`.
Read the matching file when working on a known stack.
If no file exists for the current stack, offer to generate one (see "When to Read This File").

## Memory

No agent here has automatic cross-session memory. `~/.ai/memory/` is for manual reference
files — read on request only, no auto-injection.

---
<br>
<br>

## `~/ALWAYS.md` format *(for step 7 — save to `~/ALWAYS.md`, not to `~/AGENTS.md`)*

```markdown
## Always-Active Skills

Unlike the Skills Index below, these are read proactively at the start
of every session — not gated behind a trigger phrase match.

| Skill | File |
|---|---|
| `<id>` | `~/.ai/skills/<id>.md` |
```

List only the skills the user selected in step 7. One row per skill.