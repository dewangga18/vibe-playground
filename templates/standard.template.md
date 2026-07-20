# Standard Template — Instructions for the Agent

Generate a coding standards file for a specific tech-stack and save it to `~/.ai/standards/<tech-stack>.md`.

## Steps

**1. Detect the stack**
- Manual trigger: user names the stack (e.g. "buat standar untuk React").
- Auto trigger: scan the codebase for stack indicators (`package.json`, `go.mod`, `Cargo.toml`, `requirements.txt`, `pyproject.toml`, etc.).
- If uncertain, ask the user to confirm before proceeding.

**2. Check for existing file**
- If `~/.ai/standards/<tech-stack>.md` already exists, show the current content and ask:
  > "File already exists. Overwrite with fresh best practices from the web?"
- Wait for explicit confirmation before continuing.

**3. Search best practices**
- Query the web for current best practices (e.g. `"<stack> best practices <current year> do dont"`).
- Prioritize official docs, widely-adopted style guides, and community consensus.
- Discard opinions without broad adoption.

**4. Generate the file**

Save to `~/.ai/standards/<tech-stack>.md` using this format as a starting structure —
adapt sections to what's most relevant for the stack. Keep it concise — aim for under 50 lines.
Expand only where the stack genuinely requires depth; no padding.

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

# Standards — \<Tech Stack>

> One sentence: what this stack is and when to use it.

## Do
- ...

## Don't
- ...

## Project Structure
- ...

## Naming Conventions
- ...

## \<Agent-chosen section>
Pick the single most relevant section for this stack. Examples:
- **Error Handling** — Go, Rust, Node.js
- **State Management** — React, Vue, Flutter
- **Security** — Express, Django, Rails
- **Performance** — SQL, Redis, C++
- **Testing** — any stack where testing patterns are non-obvious
- ...
