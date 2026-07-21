---
name: report-by-git-changes
description: Generate concise changelog-style reports from current git changes.
---

# Report By Changes Skill

## Input and Source Discovery

Priority order:
1. **Explicit `commit` param** → use directly, ignore working directory.
   - One commit: `git diff <id>^ <id>` (or `git show <id>` for root commit).
   - Two commits: `git diff <id1> <id2>`.
2. **No param, uncommitted changes** (`git status --porcelain` non-empty) → use:
   - `git diff HEAD` (staged + unstaged)
   - Plus untracked: `git status --porcelain | grep '^??'` (invisible to `git diff`)
3. **Clean working dir, no param** → latest commit: `git diff HEAD^ HEAD` (or `git show HEAD` for repo's only commit).

Read affected files only when additional context needed beyond diff.

# Locale Option

`locale=<locale>` — Any ISO 639-1 code (en, id, ja, fr...). Default: `en`.

Agent adapts output wording AND internal working process (verb forms, changelog patterns, grouping labels) to match locale. Use natural changelog register — not word-for-word translation.

# Size Option

`size=<number>` (50, 70, 80, 90, 100...) — target line length in chars. Default: 65–100.
- `size=80` → lines ~80 chars
- `size=90-100` → lines 90–100 chars


# Output Format

Markdown bullet list. One change per line. Short, consistent, product-oriented. Avoid raw technical diff.

Examples:

`locale=en`:
```md
- Refactored network error handling mechanism.
- Cleaned up error handling boilerplate in the repository layer.
- Added account not found error handling during OTP verification.
- Improved application localization and error messages.
- Updated reward search thumbnail rendering.
```

`locale=id`:
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

**Grouping:** Summarize by concern, not file. (Bad: `Updated NetworkManager.swift.` Good: `Refactored network error handling mechanism.`)

**Technical terms:** Include only when meaningful (`SDK wrapper`, `OTP`, `Swift Concurrency`). (Bad: `Removed #if targetEnvironment(simulator).` Good: `Simplified SDK integration using centralized wrapper.`)

## Transformation Example

Input:
```
Create SDK wrapper.
Remove simulator conditional compilation.
Fix adaptive image layout.
Fix redeem availability checking.
Investigate scanner offline issue.
```

Output (`locale=en`):
```md
- Implemented centralized SDK wrapper to simplify integration and reduce conditional code.
- Cleaned up image layout after removing adaptive image handling.
- Fixed available redeem logic.
- Fixed scanner behavior in offline mode.
```

# Quality Checklist

Before generating:
* One line per change
* No detailed explanation
* No file names
* No commit message format
* Uses changelog style
* Groups related changes
* Mentions important technical concepts only
* Output language matches the requested locale
* Diff source resolved correctly (explicit `commit` param > uncommitted changes > latest commit)