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

---

# Language Option

Supported languages:

```bash
language=id
language=en
```

Default:

```bash
language=id
```

---

# Output Format

Output only markdown bullet list.

Example:

```md
- Refaktorisasi mekanisme error handling pada layanan jaringan.
- Pembersihan boilerplate error handling pada layer repository.
- Penambahan penanganan error akun tidak ditemukan pada verifikasi OTP.
- Penyempurnaan lokalisasi dan pesan error aplikasi.
- Penyesuaian tampilan thumbnail pada fitur pencarian reward.
```

---

# Writing Style Rules

## Indonesian

Use concise changelog wording.

### Feature / Addition

Patterns:

```
Penambahan <fitur/perubahan>.
Penerapan <mekanisme> untuk <tujuan>.
```

Examples:

```
Penambahan penanganan error akun tidak ditemukan pada verifikasi OTP.
Penerapan wrapper SDK terpusat untuk menyederhanakan integrasi lintas platform.
```

---

### Improvement

Patterns:

```
Penyempurnaan <area>.
Penyesuaian <area>.
Perbaikan <area>.
Optimasi <area>.
```

Examples:

```
Penyempurnaan pesan dan terjemahan error aplikasi.
Penyesuaian tampilan thumbnail pada fitur pencarian reward.
Perbaikan logika available redeem.
```

---

### Refactor

Patterns:

```
Refaktorisasi <area>.
Penyederhanaan <area>.
Pembersihan <area>.
```

Examples:

```
Refaktorisasi mekanisme error handling pada layanan jaringan.
Penyederhanaan integrasi SDK melalui wrapper terpusat.
Pembersihan kode kondisional yang tidak diperlukan.
```

---

## English

Use concise release note wording.

### Feature

Patterns:

```
Added <feature>.
Implemented <mechanism> for <purpose>.
```

Examples:

```
Added account not found error handling during OTP verification.
Implemented centralized SDK wrapper to simplify cross-platform integration.
```

---

### Improvement

Patterns:

```
Improved <area>.
Updated <area>.
Optimized <area>.
```

Examples:

```
Improved reward search thumbnail rendering.
Updated application localization and error messages.
```

---

### Refactor

Patterns:

```
Refactored <area>.
Simplified <area>.
Cleaned up <area>.
```

Examples:

```
Refactored network error handling mechanism.
Simplified SDK integration using centralized wrapper.
```

---

# Grouping Rules

Group related changes into meaningful summaries.

Do not output file-based changes:

Bad:

```
- Updated NetworkManager.swift.
- Updated Repository.swift.
- Updated ViewModel.swift.
```

Good:

```
- Refaktorisasi mekanisme error handling pada layanan jaringan.
```

---

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

Good:

```
- Penyederhanaan integrasi SDK melalui wrapper terpusat serta pengurangan kode kondisional.
```

---

# Transformation Example

## Input

```
Create SDK wrapper.
Remove simulator conditional compilation.
Fix adaptive image layout.
Fix redeem availability checking.
Investigate scanner offline issue.
```

## Output

```md
- Penerapan wrapper SDK terpusat untuk menyederhanakan integrasi dan mengurangi kode kondisional.
- Perapihan tampilan gambar setelah penghapusan adaptive image.
- Perbaikan logika available redeem.
- Perbaikan proses scan pada kondisi offline.
```

---

# Quality Checklist

Before generating final output:

* One line per change
* No detailed explanation
* No file names
* No commit message format
* Uses changelog style
* Groups related changes
* Mentions important technical concepts only
* Matches requested language

```
```

