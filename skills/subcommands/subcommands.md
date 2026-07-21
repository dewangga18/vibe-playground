---
name: subcommands
description: Sync skills to native custom commands via pointer files.
---

# Subcommands Sync

Meta-skill. Doesn't do task work itself — keeps native custom commands in
Kiro/OpenCode/Freebuff pointing at the single source of truth in
`~/.ai/skills/`.

## Params

```
-path=<dir>       Scan a custom skills directory instead of ~/.ai/skills/
-github=<url>     Fetch a single skill file from a GitHub raw URL
-md=<markdown>    Use pasted markdown content as the skill source
```

All params optional. Default: scan `~/.ai/skills/`. Each param adds one skill to sync queue then falls back to default scan. Multiple params can be combined.

## Canonical identity rule

**Skill identity = filename, not frontmatter.**
- `~/.ai/skills/grademe.md` → id = `grademe`
- Frontmatter `name:`/`description:` may be missing/inconsistent — never trust as primary key for Kiro or OpenCode.
- **Exception:** Freebuff requires frontmatter `name:` (its own requirement, not our choice).

## When to stop and ask

**Pause + get user approval** before writing/deleting/overwriting when:
- **Case not explicitly covered** — e.g. partial agent match, scan error, broken frontmatter, agent behavior outside General fallback.
- **Multiple valid options exist** — e.g. two plausible targets, id collision, agent with two command styles.

**Action:** State what was found, list option(s), wait for answer. Never silently pick the safest-looking option. Applies at every step.

## Step 1 — Discover

Resolve skill source(s) via priority chain. All resolved skills feed into single list for Step 2+.

### Mode A — Default: scan `~/.ai/skills/`

No params, or fallback after failed mode.

List `~/.ai/skills/*.md`. Per file:
- `id` = filename without `.md`
- `description` = frontmatter `description:` if present (never fabricate)

Directory missing/empty → stop, inform user, ask to populate or provide source via params.

### Mode B — Custom path: `-path=<dir>`

Scan `<dir>/*.md` instead. Same id/description extraction as Mode A.

**Validation:** Directory must exist, be readable, contain ≥1 `.md` file. Fail → report error, fall back to Mode A.

### Mode C — GitHub fetch: `-github=<url>`

Fetch single skill from GitHub raw URL. No clone needed.

URL format: `https://raw.githubusercontent.com/<user>/<repo>/<branch>/path/to/<name>.md`

**Steps:**
1. Validate: URL ends in `.md`, HTTP 200. Fail → report, fall back to Mode A.
2. Fetch content.
3. `id` = filename from URL (last segment, no `.md`).
4. `description` = frontmatter `description:` if present.
5. Ask: "Save fetched skill as `~/.ai/skills/<id>.md`?"
   - Yes → write file, include in sync queue.
   - No → in-memory only (note in Step 6 report).

After `-github=` skill, continue with Mode A for remaining skills.

### Mode D — Pasted markdown: `-md=<content>`

User provides raw markdown inline.

**Steps:**
1. Frontmatter present? Use `name:` as `id` + `description:` as description.
2. Frontmatter absent or `name:` missing? `id` = first `# Heading` (slugified: lowercase, spaces→`-`, strip special chars). Description blank.
3. Confirm resolved `id` with user.
4. Ask: "Save as `~/.ai/skills/<id>.md`?" Same handle as Mode C (write vs in-memory).

After `-md=` skill, continue with Mode A for remaining.


## Step 2 — Detect target agent

Check cwd for markers (order matters):
- `.kiro/` → **Kiro**
- `.opencode/` or `opencode.json` → **OpenCode**
- `.agents/` → **Freebuff**
- None → **General / unlisted agent** (fallback in Step 5)

Multiple markers? Ask user which to sync. None match? You already know which agent you *are* — proceed to General fallback.

## Step 3 — Scan what's already integrated

Detection differs per agent (not optional — each works differently):

**Kiro** (global, home dir):
```
grep -oP '(?<=~/.ai/skills/)[\\w-]+(?=\\.md)' ~/.kiro/prompts/*.md
```
Also check `@id` registrations from `/prompts create`. If id appears as BOTH file in `~/.kiro/prompts/` AND `/prompts create` entry → flag as duplicate to clean up.

**OpenCode** (global, home dir + XDG fallback):
```
grep -oP '(?<=~/.ai/skills/)[\\w-]+(?=\\.md)' ~/.opencode/commands/*.md ~/.config/opencode/commands/*.md 2>/dev/null
```
Global, same as Kiro.

Scan errors (permission denied, missing dir, malformed file) → **don't** treat as "zero integrated". Report error and ask.

**Freebuff** (global `~/.agents/skills/` only):
Global, all repos. Single source of truth — no per-repo integration.
```
for d in ~/.agents/skills/*/; do
  [ -d "$d" ] || continue
  name=$(basename "$d")
  target=$(readlink "${d}SKILL.md" 2>/dev/null)
  [ "$target" = "$HOME/.ai/skills/${name}.md" ] && echo "$name"
done
```

**General / unlisted agent:**
No known file location. Check:
1. Does agent have a custom-command mechanism? Check `--help`, docs, config for "custom command", "slash command", etc.
2. Has any `~/.ai/skills/` id been registered? Ask agent's tooling to list commands, check each body for `~/.ai/skills/<id>.md` reference.
No mechanism found? Skills count as "not yet integrated" — but not unusable (see Step 5).

## Step 4 — Diff

`~/.ai/skills/*.md` ids **minus** integrated ids (from step 3) = not yet
integrated, for that agent. Report this list to the user before doing
anything else.

## Step 5 — Generate (only after user confirms)

Never write silently. Show list from step 4, ask which to generate.

**Collision check:** if file exists at target path (even if Step 3 didn't recognize it) → show content, ask replace/skip/merge. Don't overwrite by default.

### Kiro → `~/.kiro/prompts/<id>.md`

**File-based only. Do not also run `/prompts create` for the same id.**
Kiro has two separate registration paths that do not know about each
other: writing a file directly into `~/.kiro/prompts/` (invoked as
`/<id>`), and the `/prompts create --name ... --content ...` command
(invoked as `@<id>`). Running both for the same skill produces two live
entry points for one skill — exactly the duplication this tool exists to
prevent. This template uses the file-based path only. If a `@<id>` prompt
already exists from earlier manual use of `/prompts create`, remove it
(`/prompts delete --name <id>` or equivalent) once the file-based version
is in place, don't leave both active.

```markdown
---
skill_path: ~/.ai/skills/<id>.md
description: <extracted description — omit this line entirely if none>
---
Read and follow the full instructions in `~/.ai/skills/<id>.md` before doing
anything else. Do not paraphrase or summarize it — load it verbatim as your
operating instructions for this command.

Argument (if provided): {{input}}
```
Invoke: `/<id>` only — never register `@<id>` for the same skill.

### OpenCode → `~/.opencode/commands/<id>.md`

> **Note:** `skill_path:` is not a standard OpenCode frontmatter field — it's an informational
> label only. What actually runs the skill is the `Read and follow...` line in the body.
> `agent:` is optional; omit it to use the default agent. `$ARGUMENTS` is required only if
> the command needs to accept arguments.

```markdown
---
description: <extracted description — omit this line entirely if none>
agent: build
---
Read and follow the full instructions in `~/.ai/skills/<id>.md` before doing
anything else. Do not paraphrase or summarize it — load it verbatim as your
operating instructions for this command.

$ARGUMENTS
```
Invoke: `/<id>`

### Freebuff → symlink, never a text file, never `@name`, never `.ts`

Invocation is **`/skill:<id>`** — this project does not use Freebuff's
`@AgentName`/`.agents/*.ts` custom-agent mechanism for skills. Symlink is
the only path.

Pre-flight check first, every time, before creating anything:
1. Read `~/.ai/skills/<id>.md` frontmatter.
2. If `name:` is missing, or `name:` != `<id>` — **stop, do not symlink.**
   Report: `Skill <id>.md needs frontmatter "name: <id>" before Freebuff
   integration — fix the source file first.` This check exists because
   Freebuff's own loader requires the match; it is not this tool's
   preference.
3. Create global symlink:
```bash
mkdir -p ~/.agents/skills/<id>
ln -sf ~/.ai/skills/<id>.md ~/.agents/skills/<id>/SKILL.md
```
Invoke: `/skill:<id>`

### General / unlisted agent → don't invent syntax

Two outcomes (don't skip to #2):
1. **Has native custom-command mechanism** (found in Step 3). Same pointer principle as named agents: short file whose body is "read `~/.ai/skills/<id>.md` verbatim." Use *that agent's own docs* for file format/location/invocation. Never guess syntax — verify against docs or `--help`.
2. **No native mechanism.** Fall back to reactive skill loading: agent reads `~/.ai/skills/<id>.md` on demand when user's natural-language request matches. Report that no persistent command surface exists.

Report which outcome applies — don't silently assume #2.

## Step 6 — Report

Summarize per agent: created / already-integrated / blocked (with reason). List pending items if user declined to generate all.
For `-github=` or `-md=` skills not saved to disk: note they're in-memory only (won't persist).

## Never do

- Never copy skill body content into a prompt/command file — pointer only,
  every time.
- Never fabricate `description` or `argument-hint` when the source doesn't
  have one — omit the field.
- Never assume Freebuff frontmatter is present or correct — always validate
  before symlinking, every single time, not just on first setup.
- Never treat Freebuff's integration status as per-repo — it's global at `~/.agents/skills/`, applies to all repos.
- Never invent command syntax for an unlisted agent (guessing `/name` vs
  `@name` vs anything else) — verify against that agent's own docs or
  `--help` first, or fall back to reactive skill loading instead.
- Never resolve an unhandled case, or choose between multiple valid
  options, without asking the user first — see "When to stop and ask."
