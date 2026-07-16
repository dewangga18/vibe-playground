# AGENTS.md

> Generate with `/init` or Plan Mode, then trim.
> README for agents, not humans. Don't duplicate README.md or what linters already enforce.
> Only include what an agent can't infer from the codebase itself. Keep this ~50 lines.

## Stack
**Name:** <project/app name><br>
**What it does:** <1-2 sentence context><br>
**Tech stack:** <e.g. Go 1.22, or iOS 16+ / Swift 5.9><br>
**Environments:** `Staging` <endpoint/config> · `Production` <endpoint/config><br>
**Folders Example Pattern:**
* `cmd/api/` - entry point
* `internal/` - internal packages
* `Dependencies/` - local SPM

## Commands
```bash
<install command>
<run command>
<test command>
<lint/format command>
```
Optional if you have tests:<br>
Every new function/endpoint needs a test. Run tests before considering a task done.<br>
CI config lives at `<path>`.

## Mandatory Rules
- **Verify before done.** Never mark complete without proving it works. Diff vs main when relevant.
- **Demand elegance (balanced).** For non-trivial changes, look for the cleaner solution — skip for simple, obvious fixes.
- **Autonomous bug fixing.** Given a bug report, just fix it. Check `.logs/errors/` first.
- **Security.** Never commit `.env*`/secrets. Files agents must NOT touch: `<e.g. migrations/*>`
- **Plan mode.** Tasks touching >2 files need a plan in `.tasks/todo.md` first, tracked as you go.
- **Core principles.** Simplicity first, fix root causes (no temp patches), minimal impact.

## Conventions
- Naming: <e.g. camelCase vars, PascalCase types>
- **Every new function/method needs a short doc comment** (purpose, params, return) — use the
  language's standard format (godoc, `///`, JSDoc, docstring).
- Commit format: `<type>: <short description>` — PR title: `[<project_name>] <Title>`
- Run tests + lint before committing.

## Gotchas
<non-obvious thing that trips people/agents up, and the fix>