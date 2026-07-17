---
name: subcommands
description: >
  Sync skills to native custom commands in Kiro, OpenCode, and Freebuff —
  without ever duplicating skill body content. Supports four source modes:
  default scan of ~/.ai/skills/, custom path (-path=), direct GitHub fetch
  (-github=), and pasted markdown (-md=). Discovers skills, detects which
  agent you're in, diffs against what's already integrated, and generates
  missing pointer files on confirmation. Includes a general fallback
  procedure for any other CLI agent not explicitly covered.
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

All params are optional. Default behavior (no params) scans `~/.ai/skills/`.
Each param adds one skill to the sync queue, then falls back to the default
scan for the rest. Multiple params can be combined in one invocation.

## Canonical identity rule

**Skill identity = filename, not frontmatter.** For `~/.ai/skills/grademe.md`,
the skill id is `grademe`. Frontmatter `name:`/`description:` may be missing
or inconsistent in some files — never trust it as the primary key for
Kiro or OpenCode. The one exception is Freebuff, covered below, where
frontmatter `name:` is a hard requirement of Freebuff itself, not a choice
made by this sync tool.

## When to stop and ask

This skill is not allowed to guess its way through ambiguity. Pause and
get explicit user approval before writing, deleting, or overwriting
anything whenever:

- **A case isn't explicitly covered** by this file — e.g. a directory
  structure that partially matches two known agents, a scan command that
  errors out instead of returning a clean list, a skill file with broken
  or unparseable frontmatter, an agent whose docs describe something that
  doesn't fit either outcome in the General fallback.
- **More than one valid option exists** — e.g. two plausible target
  locations, a skill id that collides with a command that already exists
  through some other mechanism, an agent that supports two different
  native command styles.

When this happens: state plainly what was found, list the option(s) —
even if only one exists and it just needs confirming — and wait for the
user's answer before taking any write action. Never silently pick the
option that looks safest or most likely; surface it instead. This applies
at every step below, not just Step 5.

## Step 1 — Discover

Resolve the skill source(s) using this priority chain. All resolved skills
feed into a single list for Step 2 onward.

### Mode A — Default: scan `~/.ai/skills/`

No params given, or as the fallback after any failed mode below.

List `~/.ai/skills/*.md`. For each file:
- `id` = filename without `.md`
- `description` = value of frontmatter `description:` if present, otherwise
  leave blank — never fabricate one.

If the directory is missing or empty → stop, inform the user, ask them to
either populate `~/.ai/skills/` or provide a source via one of the params
below.

### Mode B — Custom path: `-path=<dir>`

Scan `<dir>/*.md` instead of `~/.ai/skills/`. Apply the same id/description
extraction as Mode A.

Validation before scanning:
- Directory must exist and be readable.
- Must contain at least one `.md` file.

If validation fails → report the error and fall back to Mode A.

### Mode C — GitHub fetch: `-github=<url>`

Fetch a single skill file directly from a GitHub raw URL. No clone needed.

Expected URL format:
```
https://raw.githubusercontent.com/<user>/<repo>/<branch>/path/to/<name>.md
```

Steps:
1. Validate: URL must end in `.md` and be reachable (HTTP 200). If not →
   report error and fall back to Mode A.
2. Fetch the file content.
3. Derive `id` from the filename in the URL (last path segment, no `.md`).
4. Extract `description` from frontmatter if present; otherwise leave blank.
5. Ask user: *"Save fetched skill as `~/.ai/skills/<id>.md`?"*
   - If yes → write file, then include `<id>` in the sync queue.
   - If no → include `<id>` in the sync queue using the fetched content
     in-memory only (not saved to disk). Note this in the Step 6 report.

After processing the `-github=` skill, continue with Mode A for remaining
skills already in `~/.ai/skills/`.

### Mode D — Pasted markdown: `-md=<content>`

User provides raw markdown content inline.

Steps:
1. If frontmatter is present:
   - Use `name:` field as `id` (if present and valid).
   - Use `description:` field as description (if present).
2. If frontmatter is absent or `name:` is missing:
   - Use the first `# Heading` as `id` (slugified: lowercase, spaces → `-`,
     strip special chars).
   - Leave description blank.
3. Ask user to confirm the resolved `id` before proceeding.
4. Ask user: *"Save as `~/.ai/skills/<id>.md`?"*
   - If yes → write file, then include `<id>` in the sync queue.
   - If no → include in sync queue using in-memory content only.

After processing the `-md=` skill, continue with Mode A for remaining
skills already in `~/.ai/skills/`.

---

## Step 2 — Detect target agent

Check cwd for markers, in this order:
- `.kiro/` present → **Kiro**
- `.opencode/` or `opencode.json` present → **OpenCode**
- `.agents/` present (and not matched above) → **Freebuff**
- None of the above → **General / unlisted agent**, follow the fallback
  procedure in Step 5 instead of the three named ones.

If multiple markers exist, ask the user which one to sync for — don't
guess. If none match, don't just stop and ask "which agent?" either; you
already know which agent you *are* (you're running inside it) — proceed to
the General fallback below, which works for any agent without needing a
name-specific branch.

## Step 3 — Scan what's already integrated

Detection method is different per agent — this is not optional, each one
genuinely works differently:

**Kiro** (global, home dir):
```
grep -oP '(?<=~/.ai/skills/)[\w-]+(?=\.md)' ~/.kiro/prompts/*.md
```
Any id found here is already integrated, globally, for every repo.

Also check for leftover `@id`-style registrations from `/prompts create`
(run `/prompts list` or equivalent). If an id shows up both as a file in
`~/.kiro/prompts/` **and** as a `/prompts create` entry, flag it as a
duplicate to clean up — don't just report it as "integrated" and move on.

**OpenCode** (global, home dir — also check the XDG fallback location):
```
grep -oP '(?<=~/.ai/skills/)[\w-]+(?=\.md)' ~/.opencode/commands/*.md ~/.config/opencode/commands/*.md 2>/dev/null
```
Same as Kiro: global, applies to every repo.

If any scan command errors (permission denied, directory missing in an
unexpected way, malformed file that can't be grepped cleanly) — don't
treat that as "zero results, nothing integrated." That's an unhandled
case per the section above; report the error and ask before assuming
anything about integration status.

**Freebuff** (per-repo, cwd — not global, do not skip this distinction):
```
for d in .agents/skills/*/; do
  name=$(basename "$d")
  target=$(readlink "${d}SKILL.md" 2>/dev/null)
  [ "$target" = "$HOME/.ai/skills/${name}.md" ] && echo "$name"
done
```
An id integrated in one repo tells you nothing about any other repo.
Freebuff status must be re-checked every time you're in a different
project.

**General / unlisted agent:**
There's no known file location to grep yet, so check for evidence in two
places instead:
1. Does the agent have its own documented custom-command mechanism at
   all? Check its own `--help`, docs, or config schema for terms like
   "custom command", "slash command", "prompt template", "custom agent".
2. If yes, has it already been used for any `~/.ai/skills/` id? Search
   wherever that mechanism stores its definitions (ask the agent's own
   tooling to list existing commands, then check each one's body for a
   `~/.ai/skills/<id>.md` reference — same idea as the grep used for
   Kiro/OpenCode, just against an unknown file layout).
If there's no discoverable custom-command mechanism at all, every skill in
`~/.ai/skills/` counts as "not yet integrated as a command" — but that
doesn't mean unusable, see Step 5.

## Step 4 — Diff

`~/.ai/skills/*.md` ids **minus** integrated ids (from step 3) = not yet
integrated, for that agent. Report this list to the user before doing
anything else.

## Step 5 — Generate (only after user confirms)

Never write files silently. Show the list from step 4, ask which ones (or
"all"), then generate:

Before writing any file: if a file already exists at that exact path
(even if Step 3 didn't recognize it as a skill pointer — e.g. it's a
hand-written command with different content), that's a collision, not a
routine overwrite. Show the existing content, ask whether to replace it,
skip it, or merge — don't overwrite by default just because generation
was approved for the id in general.

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
3. Only if valid:
```bash
mkdir -p .agents/skills/<id>
ln -sf ~/.ai/skills/<id>.md .agents/skills/<id>/SKILL.md
```
Invoke: `/skill:<id>`

### General / unlisted agent → figure out the mechanism, don't invent syntax

Two possible outcomes, don't skip straight to the second:

1. **The agent has some native custom-command mechanism** (found in Step
   3). Follow the exact same pointer principle as the three named agents —
   a short file/entry whose entire body is "read `~/.ai/skills/<id>.md`
   and follow it verbatim, do not paraphrase," using whatever file format,
   location, and invocation syntax *that agent's own documentation*
   specifies. Never guess the syntax (`/name` vs `@name` vs something
   else) — verify against that agent's actual docs or `--help` output
   first, the same way the Freebuff `.ts` assumption earlier turned out
   wrong until checked against real behavior.

2. **No native custom-command mechanism exists.** Don't force one. Fall
   back to reactive skill loading instead: rely on the agent recognizing
   the user's natural-language request (e.g. "grade my session") and
   reading the matching `~/.ai/skills/<id>.md` on demand — the same
   fallback already described in the root `AGENTS.md`. Report to the user
   that this agent has no persistent command surface, so skills here are
   invoked by describing what you want, not by a fixed command name.

Either way, report back which outcome applies — don't silently assume
outcome 2 just because outcome 1 takes more digging.

## Step 6 — Report

Summarize per agent: created / already-integrated / blocked (with reason).
If the user declined to generate everything, list what's still pending so
it's easy to re-run later.
For skills added via `-github=` or `-md=` that were not saved to disk,
note that they are in-memory only and will not persist across sessions.

## Never do

- Never copy skill body content into a prompt/command file — pointer only,
  every time.
- Never fabricate `description` or `argument-hint` when the source doesn't
  have one — omit the field.
- Never assume Freebuff frontmatter is present or correct — always validate
  before symlinking, every single time, not just on first setup.
- Never treat Freebuff's integration status as global — it's per-repo,
  re-check on every cwd change.
- Never invent command syntax for an unlisted agent (guessing `/name` vs
  `@name` vs anything else) — verify against that agent's own docs or
  `--help` first, or fall back to reactive skill loading instead.
- Never resolve an unhandled case, or choose between multiple valid
  options, without asking the user first — see "When to stop and ask."
