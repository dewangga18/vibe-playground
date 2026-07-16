# AGENTS.local Template — Instructions for the Agent

Generate the local project agent instructions file at `./AGENTS.md` (root of the current project).

## Steps

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

---
<br>
<br>

# AGENTS.md

> Global config, skills & standards: `~/AGENTS.md` — read it when you need skills or stack standards.
> This file is project-specific context only. Don't duplicate what linters or the codebase already enforce.

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
