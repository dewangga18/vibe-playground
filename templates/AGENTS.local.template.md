# AGENTS.local Template — Instructions for the Agent

Generate the local project agent instructions file at `./AGENTS.md` (root of the current project).

## Steps

**0. Check for global toolkit**
- Check if `~/.ai/` exists and your global agent instructions file (e.g. `~/AGENTS.md` or
  equivalent for your agent) exists.
- **If `~/.ai/` is missing entirely:**
  > "Global AI toolkit (`~/.ai/`) not found. This project uses `vibe-playground` for shared skills
  > and standards. You can:
  > - **Set it up now** — point me to your local `vibe-playground/` clone or share the repo URL
  >   and I'll guide you through setup.
  > - **Provide your own context** — share any plan, standards doc, or template you want me to
  >   follow for this project. I'll use that as the basis for `./AGENTS.md`.
  > - **Skip for now** — I'll generate a minimal `./AGENTS.md` from the project codebase only,
  >   no global context. You can set up the toolkit later."
  - Wait for user's choice before continuing.
  - If option 1: run setup, then continue with this template.
  - If option 2: read the provided context, then generate `./AGENTS.md` incorporating it.
    Do not reference `~/.ai/` or the global instructions file in the output.
  - If option 3: generate `./AGENTS.md` from codebase scan only.
    Do not reference `~/.ai/` or the global instructions file in the output.
- **If `~/.ai/` exists but the global instructions file is missing:**
  > "Global agent instructions file not found. Want me to generate it from
  > `~/.ai/templates/AGENTS.global.template.md` before continuing?"
  - If yes: run `AGENTS.global.template.md` first, then continue with this template.
  - If no: continue, but note that skills and adapter config won't be available this session.

**1. Detect existing file**
- Check if `./AGENTS.md` already exists. Note this as first-time build or regenerate — it
  determines whether step 4 (reconcile) runs.
- Do NOT ask about overwrite/improve yet. That decision happens in step 4, once fresh project
  info has actually been collected — asking now would be a blind guess with nothing concrete
  to compare against.

**2. Check native init capability**
- Read the active adapter's `Native Init Behavior` block (`native_init_command`,
  `native_entry_files.project`, `reconciliation_policy`). If the adapter doesn't declare this
  block, treat as no native mechanism and go straight to step 3 (manual).
- **Native entry file already has real content** (e.g. `./CLAUDE.md` exists and is not just a
  pointer) → use it as the data source for step 3. Do not re-scan the codebase from scratch;
  extract stack/commands/architecture/gotchas from it instead.
- **Native entry file is missing or is just a pointer, but `native_init_command` exists** →
  offer:
  > "This agent has `<native_init_command>` for scanning the codebase. Want me to run it first,
  > then map the output into `./AGENTS.md`?"
  - If yes: run it, then use its output as the source for step 3 (same as above).
  - If no: fall through to step 3 (manual).
- **No native mechanism declared** → go straight to step 3 (manual).

**3. Collect project info**
<br>*Populate the following either by extracting from the native source identified in step 2, or
by scanning the codebase directly if no native source is available — do not do both.*
- Name, purpose, tech stack, environments (staging/prod endpoints if visible in config).
- Key commands: install, run, test, lint.
- Top-level folder structure: scan 1-2 levels deep.
- Non-obvious gotchas: look for comments in config files, existing docs, or ask the user.
- If uncertain about stack (multiple stacks found, or native source is ambiguous), ask the
  user to confirm.

**4. Reconcile with existing file** 
<br>*(skip entirely if step 1 found no existing `./AGENTS.md`
— go straight to step 5 with a fresh build)*
- Classify each output section into two kinds:
  - **Inferable** — Stack, Commands, Folders: derived directly from the codebase/native
    source, safe to refresh automatically.
  - **Curated** — Mandatory Rules, Conventions (deviations), Gotchas: usually hand-written by
    a human; the agent cannot safely regenerate these from a scan alone.
- Diff the freshly collected info (step 2-3) against the existing file's inferable sections.
  Summarize what changed (e.g. "test command changed from `jest` to `vitest`", "new folder
  `services/` not documented yet").
- Present the user with three options:
  > "`./AGENTS.md` already exists. Based on a fresh scan, I found `<N>` changes. What would
  > you like to do?
  > - **Overwrite** — replace the whole file with a freshly generated version
  > - **Suggest improvements only** — update inferable sections that changed, leave Mandatory
  >   Rules / Conventions / Gotchas untouched, and list anything new I noticed as suggestions
  >   for you to review and add manually
  > - **Keep as is** — cancel, don't touch the file"
- Wait for explicit confirmation before continuing.
  - **Overwrite** → proceed to step 5, full regeneration.
  - **Suggest improvements only** → in step 5, edit only inferable sections that changed;
    leave curated sections untouched. For anything new found in a curated category (e.g. a
    likely gotcha), append it under a `<!-- suggested, review before keeping -->` comment
    instead of writing it in directly.
  - **Keep as is** → stop here. Do not modify `./AGENTS.md`.

**5. Generate `./AGENTS.md`**
<br>Save using the format below, applying the mode chosen in step 4 (fresh build, full overwrite,
or targeted update). Keep it ~50 lines — only what an agent can't infer from the codebase itself.

**6. Wire project-level pointer**
- Check the native entry file for this project (per adapter's `native_entry_files.project`,
  e.g. `./CLAUDE.md`).
- This file auto-loads every session in this project — creating or overwriting it needs
  confirmation first, same as the global version in `AGENTS.global.template.md` step 6.
- **Already exists as a correct pointer** → nothing to do, skip silently.
- **Doesn't exist yet** → ask for approval, then create it as a pointer-guarded file:
  > "I'd like to create `<entry file>` so `<agent>` auto-loads `./AGENTS.md` for this project.
  > It'll just be a pointer — no content lives there directly. OK to create it?"
  > "👉 Project rules: `./AGENTS.md`. Do not add content directly here — run
  > `AGENTS.local.template.md` instead."
- **Exists with real content** → that means it was already consumed as the source in step 2.
  Show what was extracted and confirm before converting it to the pointer-guarded content
  above — this replaces existing content.
- If no native mechanism is declared for this adapter → skip this step entirely.

### Tip: Custom install path

*Only relevant if `~/.ai/` is set up on this machine.*

This template uses `~/.ai/` by default. <br>
If you've set `$AI_HOME` to another location, update all
`~/.ai/` references after copying:

```bash
grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
```

---
<br>
<br>

# AGENTS.md

> This file is project-specific context only. Don't duplicate what linters, standards files,
> or the codebase already enforce.
>
> *If context is insufficient for the current task, check your global agent instructions file
> in your home directory if available, or ask the user to provide additional context.*

## Stack
**Name:** <project/app name><br>
**What it does:** <1-2 sentence context><br>
**Tech stack:** <e.g. Go 1.22, or iOS 16+ / Swift 5.9><br>
**Environments:** `Staging` <endpoint/config> · `Production` <endpoint/config><br>
**Folders:**
- `<folder>/` — <purpose>

## Commands
```bash
<install command>
<run command>
<test command>
<lint/format command>
```
CI config: `<path>` *(omit if none)*

## Mandatory Rules
- **Verify before done.** Never mark complete without proving it works.
- **Autonomous bug fixing.** Given a bug report, just fix it. Check `.logs/errors/` first.
- **Security.** Never commit `.env*`/secrets. Files agents must NOT touch: `<e.g. migrations/*>`
- **Plan mode.** Tasks touching >2 files need a plan in `.tasks/todo.md` first, tracked as you go.
- **Core principles.** Simplicity first, fix root causes (no temp patches), minimal impact.

## Conventions
<!-- If ~/.ai/standards/<stack>/index.md exists and this project follows it as-is, write only:
     "Follows ~/.ai/standards/<stack>/index.md" and stop here.
     Otherwise, list ONLY what deviates from that standard (or, if no standards file exists
     for this stack, list the actual conventions in use). Do not restate the full standard. -->
- Naming: <e.g. camelCase vars, PascalCase types>
- Doc comments: every new function/method needs a short comment (purpose, params, return) in the language's standard format.
- Commit format: `<type>: <short description>`

## Gotchas
<non-obvious thing that trips agents up, and the fix>