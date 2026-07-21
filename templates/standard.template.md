# Standard Template — Instructions for the Agent

Generate a **split** coding standards set for a specific tech-stack:
- `~/.ai/standards/<tech-stack>.md` — index file (core rules + trigger table, always read)
- `~/.ai/standards/<tech-stack>/<section>.md` — one file per topic (read only on trigger match)

## Steps

**1. Detect the stack**
- Manual trigger: user names the stack (e.g. "buat standar untuk React").
- Auto trigger: scan the codebase for stack indicators (`package.json`, `go.mod`, `Cargo.toml`, `requirements.txt`, `pyproject.toml`, etc.).
- If uncertain, ask the user to confirm before proceeding.

**2. Detect scope**
- **Stack named only** (e.g. "create standard for Go", "buat standar untuk React") → full
  scope. Step 5 generates the whole baseline (+ stack-specific sections).
- **Specific topic named** (e.g. "create testing standard for Swift", "buat DI convention untuk
  Go") → scoped to that one section, even on a first-time build with no existing index yet.
  Still create the index file (step 8), just with a single row in its Sections table — do not
  pad it out with the rest of the baseline just because the index is new.
- If ambiguous which topic is meant, ask before proceeding rather than guessing.

**3. Detect locale**
- Default: `en`. Use this unless the request signals otherwise.
- Explicit signal (e.g. the request is phrased in Indonesian, or names a language directly) →
  use that locale instead.
- If step 4 finds an existing index for this stack → match the locale already recorded there
  (see the `<!-- locale: -->` marker in the index format below), regardless of what language
  this particular request happens to be phrased in. Don't let one mixed-language request fork
  a stack's standards into two languages.
- If the detected locale conflicts with what's already recorded, ask which to use rather than
  guessing.
- This only affects prose in the generated output — bullets, descriptions, one-line summaries.
  Structural markers (`# Standards —`, `## Core Rules`, `## Sections`, `## Do`, `## Don't`,
  `## Commands`, `## Notes`, table headers, file paths, code/commands) stay exactly as written
  in the formats below regardless of locale — the agent parses them (e.g. the `## Sections`
  table check in step 4), and they need to stay consistent across every stack and language.

**4. Check for existing file**
- Check both `~/.ai/standards/<tech-stack>.md` and `~/.ai/standards/<tech-stack>/`.
- **Neither exists** → first-time build. Skip to step 5.
- **Index exists but has no `## Sections` table** (old single-file format, from before this
  split existed) → treat as legacy monolithic content. Show it and ask:
  > "Found `~/.ai/standards/<tech-stack>.md` in the old single-file format. What would you like to do?
  > - **Split it** — reorganize the current content into per-section files, no fresh web search
  > - **Regenerate fresh** — discard it, do a full search and rebuild in the new split format
  > - **Keep as is** — cancel, don't touch anything"
  - **Split it** → skip step 6 (gathering source material) entirely; use the existing content as the source
    for step 7, mapping its existing sections onto the new section files as closely as possible.
  - **Regenerate fresh** → proceed as a normal fresh build (steps 5-8).
  - **Keep as is** → stop here.
- **Split-format index already exists** (has a `## Sections` table) → show the current section
  list and ask:
  > "Standards for `<tech-stack>` already exist (`<N>` sections: `<list>`). What would you like to do?
  > - **Add section(s)** — keep everything, add new ones (tell me which topics)
  > - **Refresh a section** — re-run the web search for one existing section only, replace just that file
  > - **Regenerate all** — discard everything, full fresh rebuild
  > - **Keep as is** — cancel, don't touch anything"
  - Wait for explicit choice before continuing. Scope steps 5-8 to only what was chosen
    (e.g. "Add section(s)" only generates the new section(s); it does not touch existing files).
  - This choice and step 2's scope both narrow the same thing (which sections get touched) —
    if they conflict (e.g. step 2 said "testing only" but the user picks "Regenerate all" here),
    the more specific/recent instruction wins; when unclear, ask.

**5. Determine sections**
- If step 2 found a **specific topic** → this step is just that one section (plus, if the topic
  is `dependencies`, no baseline padding). Skip the rest of this step.
- Otherwise (**full scope**), baseline candidates — include whichever genuinely apply to this
  stack, skip what doesn't:
  - `structure` — file/folder organization, naming conventions
  - `testing` — testing patterns, frameworks, conventions
  - `dependency-injection` — DI conventions (skip for stacks with no DI concept)
  - `dependencies` — approved libraries + install/add/update commands
- Add one or two stack-specific sections beyond the baseline where the stack genuinely needs
  them — e.g. **State Management** (React/Flutter), **Concurrency** (Go/Rust/Swift),
  **Error Handling** (Go/Rust/Node.js), **Security** (Express/Django/Rails).
- If step 4 chose "Add section(s)" or "Refresh a section", this step only covers those specific
  section(s), not the full baseline.

**6. Gather source material**
- Skip entirely if step 4 chose "Split it" (existing content is the source instead).
- Check first: did the user already give rough rules/points for the section(s) in scope —
  either in this request or earlier in the conversation? If so, that's the primary source.
  Don't run it through a search-and-override; the user's explicit rule wins.
  - Still do a light, targeted search per section to fill genuine gaps the user didn't cover,
    and to sanity-check against strong, widely-adopted convention. If something conflicts,
    surface the conflict and ask rather than silently picking a side.
- If no context was given yet, ask once (scoped to whatever sections are in play):
  > "Ada gambaran kasar soal rule `<section>`-nya, atau langsung aku cari best practice terkini
  > dari web?"
  - User shares rough points → treat as user-provided context, same handling as above.
  - User says search → full web search flow, no user context to anchor against.
- When searching (whether standalone or filling gaps), search per section separately, not one
  combined query (e.g. `"<stack> testing best practices <year>"`, then `"<stack> dependency
  injection patterns"`, etc.) — a combined query returns shallow results for all of them.
- Prioritize official docs, widely-adopted style guides, and community consensus. Discard
  opinions without broad adoption.

**7. Generate section files**
- Save each to `~/.ai/standards/<tech-stack>/<section>.md` using the section
  format below, written in the locale from step 3.
- Keep each one short — aim for under 40 lines. Depth is fine when the topic needs it, but
  padding defeats the point of splitting: cheap, focused reads.

**8. Generate/update the index file**
- Save `~/.ai/standards/<tech-stack>.md` using the index format below, written in the locale
  from step 3. Update the `<!-- locale: -->` marker to match.
- **Core Rules**: 3-5 bullets max — only things so universal they apply no matter what the
  task is (e.g. required formatter, minimum language version). If nothing qualifies, omit the
  section entirely; most rules belong in a section file, gated behind a trigger, instead.
- **Sections table**: one row per file that actually exists after step 7 — for a scoped run
  (step 2), that means the table only grows by one row; don't add rows for baseline sections
  that weren't actually generated. Trigger pattern should be realistic phrases from actual
  requests (e.g. `"add a test"`, `"where should this file go"`), not the section name repeated back.

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

## Index file format — `~/.ai/standards/<tech-stack>.md`

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
| `structure` | `<example phrases>` | File/folder organization, naming | `~/.ai/standards/<tech-stack>/structure.md` |
| `testing` | `<example phrases>` | Testing conventions and patterns | `~/.ai/standards/<tech-stack>/testing.md` |
| `dependency-injection` | `<example phrases>` | DI conventions | `~/.ai/standards/<tech-stack>/dependency-injection.md` |
| `dependencies` | `<example phrases>` | Approved libraries + install/add/update commands | `~/.ai/standards/<tech-stack>/dependencies.md` |
| `<stack-specific>` | `<example phrases>` | `<description>` | `~/.ai/standards/<tech-stack>/<name>.md` |

When a task matches a trigger pattern, read that section file before responding.
Do not preload sections — except when bootstrapping a brand-new/empty project for this stack
(no existing source files yet for it, or the user is explicitly scaffolding one): read every
section file once up front in that case, since most of them apply during initial setup anyway.
```

## Section file format — `~/.ai/standards/<tech-stack>/<section>.md`

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