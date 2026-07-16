````md
# Report By Changes Skill

## Purpose

Generate concise changelog-style reports from current git changes.

The output must follow the existing release summary style:
- One change per line
- Short and consistent wording
- Product-oriented
- Avoid raw technical diff explanation

---

# Input

Analyze current git changes from the current working directory.

Use:

```bash
git status
git diff
git diff --cached
````

Read affected files only when additional context is required.

# Language Option

Accept any ISO 639-1 locale code via the `language` param:

```
language=<locale>   e.g. en, id, ja, fr, de, zh, ko, …
```

Default is **English (`en`)** unless the user explicitly passes a different locale.

The agent adapts **both output wording and its internal working process** (verb forms, changelog patterns, grouping labels) to match the requested locale. Use the natural changelog register of that language — not a literal word-for-word translation of the English patterns.


# Output Format

Output only a markdown bullet list.

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

# Writing Style Rules

These rules define the **universal changelog structure**. Apply them in whatever language the locale code represents.

Use concise changelog wording.

### Feature / Addition

Express that something new was introduced.

```
<locale-native verb for "Added/Implemented"> <what> [for <purpose>].
```

### Improvement

Express that something existing was made better.

```
<locale-native verb for "Improved/Updated/Optimized"> <area>.
```

### Refactor

Express that internals were reorganized without external behavior change.

```
<locale-native verb for "Refactored/Simplified/Cleaned up"> <area>.
```

## English Reference Patterns

Use these as the canonical baseline. Adapt verb forms for other locales.

| Category | Patterns |
|---|---|
| Feature | `Added <feature>.` · `Implemented <mechanism> for <purpose>.` |
| Improvement | `Improved <area>.` · `Updated <area>.` · `Optimized <area>.` |
| Refactor | `Refactored <area>.` · `Simplified <area>.` · `Cleaned up <area>.` |

Examples:

```
- Added account not found error handling during OTP verification.
- Implemented centralized SDK wrapper to simplify cross-platform integration.
- Improved reward search thumbnail rendering.
- Refactored network error handling mechanism.
- Simplified SDK integration using centralized wrapper.
```

# Grouping Rules

Group related changes into meaningful summaries.

Do not output file-based changes:

Bad:

```
- Updated NetworkManager.swift.
- Updated Repository.swift.
- Updated ViewModel.swift.
```

Good (`language=en`):

```
- Refactored network error handling mechanism.
```

Good (`language=id` — same concept, locale-adapted):

```
- Refaktorisasi mekanisme error handling pada layanan jaringan.
```

# Technical Detail Rules

Include technical terms only when they represent meaningful changes.

Allowed:

```
SDK wrapper
Repository
OTP
Realm migration
Swift Concurrency
API layer
```

Avoid exposing unnecessary implementation details.

Bad:

```
- Removed #if targetEnvironment(simulator).
```

Good (`language=en`):

```
- Simplified SDK integration using centralized wrapper and reduced conditional code.
```

Good (`language=id` — same concept, locale-adapted):

```
- Penyederhanaan integrasi SDK melalui wrapper terpusat serta pengurangan kode kondisional.
```

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

## Output (`language=id` — example locale)

```md
- Penerapan wrapper SDK terpusat untuk menyederhanakan integrasi dan mengurangi kode kondisional.
- Perapihan tampilan gambar setelah penghapusan adaptive image.
- Perbaikan logika available redeem.
- Perbaikan proses scan pada kondisi offline.
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
