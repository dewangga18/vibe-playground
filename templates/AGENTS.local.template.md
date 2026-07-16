# AGENTS.local Template — Instructions for the Agent

Generate the local project agent instructions file at `./AGENTS.md` (root of the current project).

## Steps

**0. Check for global toolkit**
- Check if `~/.ai/` exists and `~/AGENTS.md` exists.
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
    Do not reference `~/.ai/` or `~/AGENTS.md` in the output.
  - If option 3: generate `./AGENTS.md` from codebase scan only.
    Do not reference `~/.ai/` or `~/AGENTS.md` in the output.
- **If `~/.ai/` exists but `~/AGENTS.md` is missing:**
  > "`~/AGENTS.md` not found. Want me to generate it from `~/.ai/templates/AGENTS.global.template.md`
  > before continuing?"
  - If yes: run `AGENTS.global.template.md` first, then continue with this template.
  - If no: continue, but note that skills and adapter config won't be available this session.

**1. Check for existing file**
- If `./AGENTS.md` already exists, show the current content and ask:
  > "AGENTS.md already exists. Overwrite?"
- Wait for explicit confirmation before continuing.

**2. Detect the stack**
- Scan the project root for stack indicators: `package.json`, `go.mod`, `Cargo.toml`,
  `requirements.txt`, `pyproject.toml`, `Podfile`, `pubspec.yaml`, etc.
- If uncertain or multiple stacks found, ask the user to confirm.

**3. Collect project info**
- Name, purpose, tech stack, environments (staging/prod endpoints if visible in config).
- Key commands: install, run, test, lint — read from `package.json`, `Makefile`, README, etc.
- Top-level folder structure: scan 1-2 levels deep.
- Non-obvious gotchas: look for comments in config files, existing docs, or ask the user.

**4. Generate `./AGENTS.md`**

Save using the format below. Keep it ~50 lines — only what an agent can't infer from the codebase itself.

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

> This file is project-specific context only. Don't duplicate what linters or the codebase already enforce.
>
> *If context is insufficient for the current task, check `~/AGENTS.md` or `~/CLAUDE.md`
> if available, or ask the user to provide additional context.*

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
- Naming: <e.g. camelCase vars, PascalCase types>
- Doc comments: every new function/method needs a short comment (purpose, params, return) in the language's standard format.
- Commit format: `<type>: <short description>`

## Gotchas
<non-obvious thing that trips agents up, and the fix>