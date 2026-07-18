# Report By Changes Skill

This skill generates concise changelog-style reports from current Git changes.

## How It Works

The skill determines the diff source using this priority order:

1. **Explicit `commit` param given** → use it directly, ignore working directory state.
   - One commit-id → diff that commit against its parent: `git diff <id>^ <id>`
     (if it has no parent — root commit — fall back to `git show <id>`).
   - Two commit-ids → `git diff <id1> <id2>`.

2. **No param, working directory has uncommitted changes** → check with
   → status --porcelain`; if non-empty, use those changes:
   - `git diff HEAD` (covers staged + unstaged together)
   - plus untracked files: `git status --porcelain | grep '^??'` 
     (these are invisible to `git diff` and must be read directly)

3. **Working directory clean, no param** → fall back to the latest commit:
   `git diff HEAD^ HEAD` (or `git show HEAD` if it's the repo's only commit).

Read affected files only when additional context is needed beyond the diff.

## Locale Option

Accept any ISO 639-1 locale code via the `locale` parameter:

```
locale=<locale>   e.g. en, id, ja, fr, de, zh, ko, …
```

Default is **English (`en`)** unless the user explicitly passes a different locale.

The agent adapts **both output wording and its internal working process** (verb forms, changelog patterns, grouping labels) to match the requested locale. Use the natural changelog register of that language — not a literal word-for-word translation of the English patterns.

## Size Option

Control approximate line length (characters per bullet point) via the `size` param:

```
size=<number>   e.g. 50, 70, 80, 90, 100
```

Default is **65–100 characters** per line (current behavior). The agent will regenerate output to meet the target:
- `size=80` → each line close to or concise ~80 chars lines
- `size=90-100` → each line in 90–100 char range

## Output Format

The output must follow the existing release summary style:
- Markdown bullet list
- One change per line
- Short and consistent wording
- Product-oriented
- Avoid raw technical diff explanation

### Example (default — `locale=en`):
```markdown
- Refactored network error handling mechanism.
- Cleaned up error handling boilerplate in the repository layer.
- Added account not found error handling during OTP verification.
- Improved application localization and error messages.
- Updated reward search thumbnail rendering.
```

### Example (`locale=id`):
```markdown
- Refaktorisasi mekanisme error handling pada layanan jaringan.
- Pembersihan boilerplate error handling pada layer repository.
- Penambahan penanganan error akun tidak ditemukan pada verifikasi OTP.
- Penyempurnaan lokalisasi dan pesan error aplikasi.
- Penyesuaian tampilan thumbnail pada fitur pencarian reward.
```

## Output Rules

- Use concise changelog wording. Apply locale-native verb forms — not word-for-word translations.
- **Grouping** — summarize by concern, not by file.
  - Bad: `Updated NetworkManager.swift.`
  - Good: `Refactored network error handling mechanism.`
- **Technical terms** — include only when they represent meaningful changes (`SDK wrapper`, `OTP`, `Realm migration`, `Swift Concurrency`, `API layer`).
  - Bad: `Removed #if targetEnvironment(simulator).`
  - Good: `Simplified SDK integration using centralized wrapper and reduced conditional code.`

## Getting Started

Users cannot install this skill via `npx` or `npm`. To use this skill:

1. **Sync the skill to your agent using the subcommands skill** (Recommended):
   ```
   /subcommands
   ```
   This will detect your agent (Kiro, OpenCode, Freebuff, or other) and create the appropriate native command pointing to `~/.ai/skills/report-by-git-changes.md`.
   
   > **Note:** You must install the subcommands skill first. See [subcommands/README.md](../subcommands/README.md) for installation and usage.

2. Place the skill file in your skills directory:
   ```bash
   cp /path/to/vibe-playground/skills/report-by-git-changes/report-by-git-changes.md ~/.ai/skills/
   ```

3. **Guide your agent to use the skill**:
   > "Generate a changelog from my current git changes using the report-by-git-changes skill"
   
   Or specify options:
   > "Generate a changelog in Indonesian for commit abc1234"

## Usage

After syncing, invoke the skill with:
```
# Generate changelog from current changes
/report-by-git-changes

# Generate changelog in Indonesian
/report-by-git-changes locale=id

# Generate changelog with ~80 chars per line
/report-by-git-changes size=80

# Generate changelog with 90-100 chars per line
/report-by-git-changes size=95

# Generate changelog for a specific commit
/report-by-git-changes commit=<commit-id>

# Generate changelog between two commits
/report-by-git-changes commit=<commit-id1>..<commit-id2>

# Combine options
/report-by-git-changes locale=id size=80 commit=<commit-id>
```