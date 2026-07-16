---
name: report-by-git-changes
description: Generate concise changelog-style reports from current git changes.
---

# Report By Changes Skill

## Input and Source Discovery

Determine the diff source using this priority order:

1. **Explicit `commit` param given** → use it directly, ignore working directory state.
   - One commit-id → diff that commit against its parent: `git diff <id>^ <id>`
     (if it has no parent — root commit — fall back to `git show <id>`).
   - Two commit-ids → `git diff <id1> <id2>`.
2. **No param, working directory has uncommitted changes** → check with
   `git status --porcelain`; if non-empty, use those changes:
   - `git diff HEAD` (covers staged + unstaged together)
   - plus untracked files: `git status --porcelain | grep '^??'`
     (these are invisible to `git diff` and must be read directly)
3. **Working directory clean, no param** → fall back to the latest commit:
   `git diff HEAD^ HEAD` (or `git show HEAD` if it's the repo's only commit).

Read affected files only when additional context is required beyond the diff.

# Language Option

Accept any ISO 639-1 locale code via the `language` param:

```
language=<locale>   e.g. en, id, ja, fr, de, zh, ko, …
```

Default is **English (`en`)** unless the user explicitly passes a different locale.

The agent adapts **both output wording and its internal working process** (verb forms, changelog patterns, grouping labels) to match the requested locale. Use the natural changelog register of that language — not a literal word-for-word translation of the English patterns.


# Output Format

The output must follow the existing release summary style:
- Markdown bullet list
- One change per line
- Short and consistent wording
- Product-oriented
- Avoid raw technical diff explanation

Example (default — `language=en`):

```md
- Refactored network error handling mechanism.
- Cleaned up error handling boilerplate in the repository layer.
- Added account not found error handling during OTP verification.
- Improved application localization and error messages.
- Updated reward search thumbnail rendering.
```

Example (`language=id`):

```md
- Refaktorisasi mekanisme error handling pada layanan jaringan.
- Pembersihan boilerplate error handling pada layer repository.
- Penambahan penanganan error akun tidak ditemukan pada verifikasi OTP.
- Penyempurnaan lokalisasi dan pesan error aplikasi.
- Penyesuaian tampilan thumbnail pada fitur pencarian reward.
```

---

# Output Rules

Use concise changelog wording. Apply locale-native verb forms — not word-for-word translations.

| Category | English patterns (baseline) |
|---|---|
| Feature | `Added <feature>.` · `Implemented <mechanism> for <purpose>.` |
| Improvement | `Improved <area>.` · `Updated <area>.` · `Optimized <area>.` |
| Refactor | `Refactored <area>.` · `Simplified <area>.` · `Cleaned up <area>.` |

**Grouping** — summarize by concern, not by file. <br>
Bad: `Updated NetworkManager.swift.` <br>
Good: `Refactored network error handling mechanism.`

**Technical terms** — include only when they represent meaningful changes (`SDK wrapper`, `OTP`, `Realm migration`, `Swift Concurrency`, `API layer`). <br>
Bad: `Removed #if targetEnvironment(simulator).` <br>
Good: `Simplified SDK integration using centralized wrapper and reduced conditional code.`

# Transformation Example

## Input

```
Create SDK wrapper.
Remove simulator conditional compilation.
Fix adaptive image layout.
Fix redeem availability checking.
Investigate scanner offline issue.
```

## Output (`language=en` — default)

```md
- Implemented centralized SDK wrapper to simplify integration and reduce conditional code.
- Cleaned up image layout after removing adaptive image handling.
- Fixed available redeem logic.
- Fixed scanner behavior in offline mode.
```

# Quality Checklist

Before generating final output:

* One line per change
* No detailed explanation
* No file names
* No commit message format
* Uses changelog style
* Groups related changes
* Mentions important technical concepts only
* Output language matches the requested locale
* Diff source resolved correctly (explicit `commit` param > uncommitted changes > latest commit)