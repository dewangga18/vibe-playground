# AGENTS.local Template — Instructions for the Agent

Generate the local project agent instructions file at `./AGENTS.md` (root of the current project).

## Steps

**0. Check for global toolkit**
- Check `~/.ai/` and global agent instructions file (e.g. `~/AGENTS.md`) exist.
- **`~/.ai/` missing entirely:**
  > "Global AI toolkit not found. Options: **Set up now** (point to `vibe-playground/`) · **Provide your own context** (I'll use that) · **Skip** (generate from codebase only, no global context)"
  Wait for choice. Option 1: run setup then continue. Options 2/3: generate `./AGENTS.md` without referencing `~/.ai/`.
- **`~/.ai/` exists but global instructions missing:**
  > "Global agent instructions not found. Generate from `~/.ai/templates/AGENTS.global.template.md` first?"
  If yes: run global template first. If no: continue without skills/adapter config.

**1. Detect existing file**
- Check if `./AGENTS.md` exists. First-time build or regenerate — determines step 4.
- Do NOT ask about overwrite yet (deferred to step 4 after fresh data collected).

**2. Check native init capability**
- Read adapter's `Native Init Behavior` (`native_init_command`, `native_entry_files.project`, `reconciliation_policy`). If block missing → go to step 3 (manual).
- **Native entry file exists with real content** (e.g. `./CLAUDE.md` has content, not just a pointer) → use as data source for step 3. Don't re-scan codebase.
- **Native entry file missing/pointer-only, but `native_init_command` exists** → offer:
  > "This agent has `<native_init_command>`. Run it first, then map output into `./AGENTS.md`?"
  If yes: run, use output as source. If no: fall through to step 3 (manual).
- **No native mechanism** → go to step 3 (manual).

**3. Collect project info**
<br>*Populate from native source (step 2) or scan codebase directly — do not do both.*
- Name, purpose, tech stack, environments (endpoints from config).
- Key commands: install, run, test, lint.
- Top-level folder structure (1-2 levels deep).
- Non-obvious gotchas (config comments, existing docs, or ask user).
- If stack is ambiguous, ask user to confirm.

**4. Reconcile with existing file** *(skip if step 1 found no existing `./AGENTS.md` — go to step 5)*
- Classify output sections:
  - **Inferable** — Stack, Commands, Folders (safe to auto-refresh from codebase).
  - **Curated** — Mandatory Rules, Conventions, Gotchas (hand-written; don't auto-regenerate).
- Diff fresh info vs existing inferable sections. Summarize changes.
- Present options:
  > "`./AGENTS.md` already exists. Found `<N>` changes. What would you like to do?
  > - **Overwrite** — replace whole file
  > - **Suggest improvements only** — update inferable sections, leave curated untouched; append new findings as `<!-- suggested, review before keeping -->`
  > - **Keep as is** — cancel, don't touch the file"
- Wait for confirmation. **Keep as is** → stop.

**5. Generate `./AGENTS.md`**
<br>Save using the format below, applying the mode from step 4. Keep ~50 lines — only what agent can't infer from codebase.

**6. Wire project-level pointer**
- Check native entry file for this project (`native_entry_files.project`, e.g. `./CLAUDE.md`).
- **Requires approval.** Same risk as `AGENTS.global.template.md` step 6. Never create silently.
- **Already exists as correct pointer** → skip.
- **Doesn't exist** → ask approval, create pointer-guarded file:
  > "I'd like to create `<entry file>` so `<agent>` auto-loads `./AGENTS.md` for this project. It'll just be a pointer. OK?"
  > "👉 Project rules: `./AGENTS.md`. Do not add content here — run `AGENTS.local.template.md` instead."
- **Exists with real content** → was already consumed as source in step 2. Show extracted content, confirm before converting to pointer.
- No native mechanism declared → skip.

> **Custom path?** If `$AI_HOME` is set and differs from `~/.ai/`, run:
> ```bash
> grep -rl '~/.ai/' "$AI_HOME" | xargs sed -i '' "s|~/.ai/|$AI_HOME/|g"
> ```

---

# AGENTS.md

> This file is project-specific context only. Don't duplicate what linters, standards files, or the codebase already enforce.
>
> *If context is insufficient, check your global agent instructions file or ask the user.*

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
<!-- If ~/.ai/standards/<stack>/index.md exists and this project follows it as-is, write only "Follows ~/.ai/standards/<stack>/index.md". Otherwise list ONLY what deviates from that standard. Do not restate the full standard. -->
- Naming: <e.g. camelCase vars, PascalCase types>
- Doc comments: every new function/method needs a short comment (purpose, params, return) in the language's standard format.
- Commit format: `<type>: <short description>`

## Gotchas
<non-obvious thing that trips agents up, and the fix>
