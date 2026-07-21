# Standard Template — Instructions for the Agent

Generate a **split** coding standards set for a specific tech-stack:
- `~/.ai/standards/<tech-stack>/index.md` — index file (core rules + trigger table, always read)
- `~/.ai/standards/<tech-stack>/parts/<section>.md` — one file per topic (read only on trigger match)

## Steps

**1. Detect the stack**
- Manual trigger: user names the stack (e.g. "buat standar untuk React").
- Auto trigger: scan codebase for stack indicators (`package.json`, `go.mod`, `Cargo.toml`, etc.).
- If uncertain, ask user to confirm.

**2. Detect scope**
- **Stack named only** → full scope. Step 5 generates whole baseline + stack-specific sections.
- **Specific topic named** (e.g. "create testing standard for Swift") → scoped to one section. Still create index (step 8) with single row in Sections table — don't pad with baseline.
- If ambiguous, ask.

**3. Detect locale**
- Default: `en`. Explicit signal (e.g. Indonesian request) → use that locale.
- Existing index for this stack → match its `<!-- locale: -->` marker, ignore request language — don't fork standards into two languages.
- Conflict? Ask.
- Locale only affects prose (bullets, descriptions). Structural markers (`# Standards —`, `## Core Rules`, `## Sections`, `## Do`, `## Don't`, `## Commands`, table headers, file paths, code) stay in English — the agent parses them.

**4. Check for existing file**
- Check `~/.ai/standards/<tech-stack>/index.md`.
- **Doesn't exist** → first-time build. Skip to step 5.
- **Exists with `## Sections` table** → index already exists. Show section list and ask:
  > "Standards for `<tech-stack>` already exist (`<N>` sections: `<list>`). What would you like to do?
  > - **Add section(s)** — keep everything, add new ones (tell me which topics)
  > - **Refresh a section** — replace just that file
  > - **Regenerate all** — discard everything, full fresh rebuild
  > - **Keep as is** — cancel"
  - Wait for choice. Scope steps 5-8 to only what was chosen.
  - If step 2's scope conflicts (e.g. "testing only" but user picks "Regenerate all"), the more specific/recent wins; when unclear, ask.
- **Exists but no `## Sections` table** → not a valid index. Ask whether to overwrite or skip.

**5. Determine sections**
- **Specific topic** (step 2) → just that one section. Skip rest.
- **Full scope** — baseline candidates (include what applies, skip what doesn't):
  - `structure` — file/folder organization, naming
  - `testing` — testing patterns, frameworks
  - `dependency-injection` — DI conventions (skip if N/A for stack)
  - `dependencies` — approved libraries + install commands
- Add 1-2 stack-specific sections where genuinely needed (State Management for React/Flutter, Concurrency for Go/Rust/Swift, Error Handling, Security).
- If step 4 chose "Add section(s)" or "Refresh", only those sections — not full baseline.

**6. Gather source material**
- Check if user already gave rules/points for the section(s) in scope. If so, that's the primary source — don't search-and-override. Still do light targeted search per section to fill gaps and sanity-check against convention. Surface conflicts instead of silently picking.
- No user context given yet? Ask once:
  > "Ada gambaran kasar soal rule `<section>`-nya, atau langsung aku cari best practice terkini dari web?"
  - Shares rough points → treat as user-provided context.
  - Says search → full web search flow.
- Search per section separately (not one combined query). Prioritize official docs and widely-adopted style guides. Discard opinions without broad adoption.

**7. Generate section files**
- Save each to `~/.ai/standards/<tech-stack>/parts/<section>.md` using the section format below, in the locale from step 3.
- Keep under 40 lines each. Depth is fine when needed; padding defeats splitting.

**8. Generate/update the index file**
- Save `~/.ai/standards/<tech-stack>/index.md` using the index format below, in the locale from step 3. Update `<!-- locale: -->`.
- **Core Rules**: 3-5 bullets max — only universally applicable rules (formatter, language version). Omit section if nothing qualifies.
- **Sections table**: one row per file that exists after step 7. Don't add rows for baseline sections not generated. Use realistic trigger phrases (`"add a test"`, not section name repeated).

> **Custom path?** If `$AI_HOME` is set and differs from `~/.ai/`, run:
> ```bash
> grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
> ```

---

## Index file format — `~/.ai/standards/<tech-stack>/index.md`

```markdown
# Standards — <Tech Stack>
<!-- locale: en -->

> One sentence: what this stack is and when to use it.

## Core Rules
<!-- 3-5 bullets max, only truly universal rules. Delete this section if none qualify. -->
- ...

## Sections
<!-- One row per generated section file. Skip rows for sections that don't apply. -->
| Section | Trigger pattern | Description | File |
|---|---|---|---|
| `structure` | `<example phrases>` | File/folder organization, naming | `~/.ai/standards/<tech-stack>/parts/structure.md` |
| `testing` | `<example phrases>` | Testing conventions and patterns | `~/.ai/standards/<tech-stack>/parts/testing.md` |
| `dependency-injection` | `<example phrases>` | DI conventions | `~/.ai/standards/<tech-stack>/parts/dependency-injection.md` |
| `dependencies` | `<example phrases>` | Approved libraries + install/add/update commands | `~/.ai/standards/<tech-stack>/parts/dependencies.md` |
| `<stack-specific>` | `<example phrases>` | `<description>` | `~/.ai/standards/<tech-stack>/parts/<name>.md` |

When a task matches a trigger pattern, read that section file before responding.
Do not preload sections — except when bootstrapping a brand-new/empty project for this stack
(no existing source files yet for it, or the user is explicitly scaffolding one): read every
section file once up front in that case, since most of them apply during initial setup anyway.
```

## Section file format — `~/.ai/standards/<tech-stack>/parts/<section>.md`

```markdown
# <Tech Stack> — <Section Title>

> One-line: what this section covers and when it applies.

## Do
- ...

## Don't
- ...

<!-- Commands block only applies to the `dependencies` section — omit elsewhere -->
## Commands
​```bash
<install command>
<add command>
<update command>
​```

<!-- Only if genuinely needed for this section — omit otherwise -->
## Notes
- ...
```
