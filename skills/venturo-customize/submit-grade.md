---
name: submit-grade
description: Upload a previously-graded vibe-coding session (produced by /grademe) to the vibescore leaderboard. Use when the user asks to submit, upload, or push their grademe score, or invokes /submit-grade.
---

# submit-grade — Leaderboard Uploader

Standalone companion to `grademe`. Reads a grading result that `grademe` already persisted to disk and uploads it to the vibescore leaderboard. This skill does **not** re-grade, does **not** re-check grading evidence, and has **no access** to any prior conversation — it only trusts the JSON file on disk, and even that gets re-validated below rather than assumed correct. The one exception: it independently re-derives `session_id` (see Step 4) rather than trusting the value already in the file, which may briefly touch the transcript path — never its content, never the `misses` evidence.

## Precondition: grademe must persist its output

This skill assumes your `grademe` skill was patched to write its final graded JSON to:

```
~/.vibescore/grades/{GRADED_AT}_{SESSION_ID}.json
```

where `{GRADED_AT}` is a sortable UTC timestamp (`YYYYMMDDTHHMMSSZ`, e.g. `20260717T153045Z`) and `{SESSION_ID}` is the same deterministic id used for leaderboard dedup. The timestamp leads the filename specifically so "newest" is a plain lexical sort — no mtime, no parsing needed. See "Patch to add to grademe" at the bottom of this file. If no such file exists yet, tell the user to run `/grademe` first — do not attempt to regrade, summarize a transcript yourself, or guess at scores.

## Params

```
/submit-grade [session_id | path-to-json]   (default: newest file in ~/.vibescore/grades/)
--upload                                     (accepted for backward compat; redundant — upload already happens automatically once credentials are set, see Step 3)
--dry-run                                    (build payload, show it, skip the POST)
--no-upload                                  (alias of --dry-run, kept for symmetry with grademe)
```

## Workflow

1. **Resolve source file.**
   - Arg is a path ending in `.json` and it exists → use it.
   - Arg is bare (no `/`) — a timestamp prefix, a session_id, or a fragment of either — glob-match it against stored filenames and take the newest match:
     ```bash
     ls ~/.vibescore/grades/*<arg>*.json 2>/dev/null | sort | tail -1
     ```
   - No arg → newest file overall. Filenames are timestamp-prefixed and zero-padded, so this is a plain lexical sort — deliberately not `ls -t`/mtime, which breaks if files get synced, copied, or touched:
     ```bash
     ls ~/.vibescore/grades/*.json 2>/dev/null | sort | tail -1
     ```
   - Nothing found / no match → tell the user to run `/grademe` first, or pass a path/id/timestamp explicitly. Do NOT guess or regrade.

2. **Load & re-validate.** Never trust a file blindly — it may have been hand-edited or come from an older grademe version.
   - `schema_version == "1.0"` present.
   - `grade` object has all contract fields: `participant`, `session_date`, `total_score`, `breakdown`, `misses`, `next_session_advice` (`prompt_analysis` optional).
   - `breakdown` values each ≤ their max (planning 20 / context 20 / decomposition 15 / delegation 15 / verification 15 / token_efficiency 10 / documentation 5) and sum to `total_score`.
   - `misses` non-empty → `total_score` ≤ 94; `misses` empty → `total_score` ≥ 95.
   - Any check fails → report exactly which check failed and stop. Do NOT silently "fix" the numbers.

3. **Check credentials.**
   - `VIBESCORE_API_URL` + `VIBESCORE_API_KEY` both set → proceed to step 4, upload happens automatically, no flag needed (`--upload` is accepted but a no-op here).
   - Either missing → show the payload that *would* be sent, skip the POST (not an error), and tell the user how to enable it: generate a token at `https://vibescore-leaderboard-sigma.vercel.app/token`, then export both env vars. Do NOT invent a URL or key.

4. **Build submission body** — contract fields only:
   - Copy everything under `grade.*` except strip `prompt_analysis` (leaderboard contract has no such field; show it to the user separately instead, same as grademe already does).
   - `session_id` — **recompute independently**, don't just trust the value stored in the file (it may be stale, hand-edited, or from an older grademe version). Same derivation grademe itself uses: if `provenance.transcript_path` still exists on disk and the transcript format has a native `sessionId` field, use that; otherwise `sha256(transcript_path + session_date)`, first 16 hex chars — using the path and date strings already in the file. The hash fallback needs no file access at all, so this step never blocks a submission.
     - Caveat: if the transcript is gone by submission time *and* the original format had a native `sessionId`, the fallback hash won't match what grademe stored at grading time — the server may then treat it as a new session instead of a dedup match. Not a failure, just a heads-up if you're pruning transcripts aggressively.
   - `participant` — pass through as-is. The server overrides identity from the API key regardless, so don't rely on this field for auth.

5. **`--dry-run` / `--no-upload`** → print the built submission body and stop here, no POST.

6. **POST** via one Bash `curl` (never print the key value):
   ```bash
   curl -sS -o /tmp/grademe_upload.json -w '%{http_code}' \
     -X POST "$VIBESCORE_API_URL/scores" \
     -H "Content-Type: application/json" \
     -H "X-API-Key: $VIBESCORE_API_KEY" \
     --data @/tmp/grademe_submission.json
   ```

7. **Report by status** (as returned, don't blindly retry):
   - `201` → "Skor terkirim ke leaderboard." + `participant` + `id` from the response body.
   - `409` → "Sesi ini sudah pernah di-upload (dedup session_id) — skor tidak digandakan."
   - `401` → "VIBESCORE_API_KEY tidak valid/absen — skor TIDAK terkirim."
   - `400` → show the `error` field from the response body (payload rejected by contract).
   - anything else / curl failure → report the code + message, don't stay silent.

---

## Patch to add to your `grademe` (v0.0.1) skill

Add as a new **Step 6**, right after "Present", to run only once validation has passed — this is the delegation that lets `submit-grade` work later, in a different turn or a different session entirely, without any access to this conversation:

```
6. **Persist for leaderboard.** After presenting the result, write it to
   `~/.vibescore/grades/{GRADED_AT}_{SESSION_ID}.json` (mkdir -p the directory if
   missing). This is a plain file write — no env vars, no network call, happens
   every time regardless of whether the user intends to upload.

   - `graded_at`: compute ONE UTC timestamp for this run, format `YYYYMMDDTHHMMSSZ`
     (e.g. `20260717T153045Z` — zero-padded, no separators, so filenames sort
     correctly newest-last with a plain `sort`). Use this exact value both as the
     filename prefix and as the `graded_at` field inside the JSON, so file and
     content can never drift apart.
   - `session_id`: unchanged derivation — transcript's own `sessionId` if the
     format has one, otherwise first 16 hex chars of
     `sha256(transcript_path + session_date)`. It's now a filename SUFFIX (for
     manual lookup/dedup-tracing) rather than the whole filename; its real job is
     still inside the JSON body, where `submit-grade` reads it for leaderboard
     dedup (server 409s on a repeat).
   - File contents:
     {
       "schema_version": "1.0",
       "session_id": "...",
       "graded_at": "20260717T153045Z",
       "grade": { ...the exact validated contract JSON from step 4, unchanged... },
       "provenance": {
         "transcript_path": "...",
         "discovered": true|false,
         "file_mtime": "...",
         "line_count": 0
       }
     }

   Behavior note: because the filename is now timestamp-led instead of
   session_id-only, re-grading the same transcript creates a NEW file each time
   instead of overwriting the old one — `~/.vibescore/grades/` becomes a local
   grading history, and `/submit-grade` with no argument always picks the most
   recent run. Not a bug; if the user wants to prune old files that's a manual
   cleanup, not something either skill should do automatically.

   Tell the user the file was saved and that `/submit-grade` can push it to the
   leaderboard whenever they're ready — grading and uploading are now two
   separate, independently-runnable steps.
```

Notes on why the file, not memory:
- Skill invocations (and especially subagents dispatched via Task/Agent) never inherit prior conversation history — they only see what's explicitly written into their prompt.
- Even within the same CLI session, calling `/submit-grade` later gives it a fresh context; nothing from the `/grademe` turn carries over automatically.
- The file is therefore the *only* reliable handoff — which is also why `submit-grade` re-validates it in step 2 instead of trusting it.
