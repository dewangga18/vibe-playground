---
name: submit-grade
description: Upload a previously-graded vibe-coding session (produced by /grademe) to the vibescore leaderboard. Use when the user asks to submit, upload, or push their grademe score, or invokes /submit-grade.
---

# submit-grade — Leaderboard Uploader

Standalone companion to `grademe`. Always reads the **newest** grading result that `grademe` persisted to disk in `~/.vibescore/grades/` and uploads it to the vibescore leaderboard. This skill accepts no arguments, paths, or flags — it only ever processes the most recent graded file. It does **not** re-grade, does **not** re-check grading evidence, and has **no access** to any prior conversation — it only trusts the JSON file on disk, and even that gets re-validated below rather than assumed correct. The one exception: it independently re-derives `session_id` (see Step 4) rather than trusting the value already in the file, which may briefly touch the transcript path — never its content, never the `misses` evidence.

## Precondition: grademe must persist its output

This skill assumes your `grademe` skill writes its final graded JSON to:

```
~/.vibescore/grades/{GRADED_AT}_{SESSION_ID}.json
```

where `{GRADED_AT}` is a sortable UTC timestamp (`YYYYMMDDTHHMMSSZ`, e.g. `20260717T153045Z`) and `{SESSION_ID}` is the same deterministic id used for leaderboard dedup. The timestamp leads the filename specifically so "newest" is a plain lexical sort — no mtime, no parsing needed. The file uses `schema_version` `"1.1"` and contains the full `grade` object (including `session_name` and `compacted`) plus a `transcript_meta` object. See the "Persist step" in `grademe.md` for the exact file structure. `submit-grade` always processes the newest graded file in that directory and accepts no arguments or paths. If no such file exists yet, tell the user to run `/grademe` first — do not attempt to regrade, summarize a transcript yourself, or guess at scores.

## Workflow

1. **Resolve source file.**
    - Always use the newest graded file in `~/.vibescore/grades/`. Filenames are timestamp-prefixed and zero-padded, so this is a plain lexical sort — deliberately not `ls -t`/mtime, which breaks if files get synced, copied, or touched:
      ```bash
      ls ~/.vibescore/grades/*.json 2>/dev/null | sort | tail -1
      ```
    - Nothing found → tell the user to run `/grademe` first. Do NOT guess or regrade.

2. **Load & re-validate.** Never trust a file blindly — it may have been hand-edited or come from an older grademe version.
    - `schema_version` present (accepts `"1.0"` or `"1.1"`; `"1.1"` additionally carries `session_name`/`compacted`/`transcript_meta`).
    - `grade` object has all contract fields: `participant`, `session_date`, `total_score`, `breakdown`, `misses`, `next_session_advice` (`prompt_analysis` optional). For `schema_version` `"1.1"`, also require `grade.session_name` (non-empty string) and `grade.compacted` (boolean).
    - `transcript_meta` object present (for `"1.1"`) with all 5 sub-fields: `line_count`, `byte_size`, `sha256_prefix`, `first_timestamp`, `last_timestamp`.
    - `breakdown` values each ≤ their max (planning 20 / context 20 / decomposition 15 / delegation 15 / verification 15 / token_efficiency 10 / documentation 5) and sum to `total_score`.
    - `misses` non-empty → `total_score` ≤ 94; `misses` empty → `total_score` ≥ 95.
    - Any check fails → report exactly which field/check failed and stop. Do NOT silently "fix" the numbers.

3. **Check credentials.**
    - `VIBESCORE_API_URL` + `VIBESCORE_API_KEY` both set → proceed to step 4; upload happens automatically.
    - Either missing → show the payload that *would* be sent, skip the POST (not an error), and tell the user how to enable it: generate a token at `https://vibescore.venturo.pro/participants/`, then export both env vars (example base URL: `https://vibescore-be.venturo.pro`). Do NOT invent a URL or key.

4. **Build submission body — programmatically, never by hand.** This is the most common cause of "curl says invalid but the payload works when I send it myself": `misses`, `next_session_advice`, and `session_name` are free text (often non-English, and often containing quotes, backslashes, or short quoted transcript citations). Retyping or reassembling that text into a bash string, heredoc, or manually-edited JSON file breaks escaping in ways that are easy to miss and hard to spot by eye. Always derive the submission file directly from the already-parsed, already-validated source object from Step 1–2 using a real JSON serializer — never by hand-typing or pasting JSON text into a shell command.

    ```bash
    python3 -c "
    import json

    with open('$SOURCE_FILE') as f:
        src = json.load(f)

    grade = dict(src['grade'])
    grade.pop('prompt_analysis', None)   # leaderboard contract has no such field

    body = grade
    body['session_id'] = '$RECOMPUTED_SESSION_ID'   # from the tiered derivation below — NOT src['session_id']

    # transcript_meta is forwarded as-is from the source file, not recomputed
    if src.get('schema_version') == '1.1':
        body['transcript_meta'] = src.get('transcript_meta')
        # session_name / compacted are already inside grade and carried over automatically

    with open('/tmp/grademe_submission.json', 'w') as f:
        json.dump(body, f, ensure_ascii=False)
    "
    ```

    - Copy everything under `grade.*` except strip `prompt_analysis` (leaderboard contract has no such field; show it to the user separately instead, same as grademe already does). `session_name`, `compacted`, and `transcript_meta` are **forwarded as-is** from the file (not recomputed) — per the decision that grademe is the single source of truth for these.
    - `session_id` — **recompute independently** (safety-check; the old architecture is retained), but the algorithm is now **aligned to the same tiered logic** grademe uses in its Persist step:
      1. Native ID per-harness (highest priority): Claude Code → transcript `sessionId` field; opencode → `s.id` from discovery; Kiro CLI → filename; Freebuff → timestamp folder name.
      2. If tier 1 unavailable → filename stem via `basename`.
      3. If tier 2 is generic/collision-prone (e.g. `"log"`) → `sha256(transcript_path + session_date)`, first 16 hex chars.
      - If the transcript is no longer on disk at submission time, skip tier 1–3 that need the file and automatically fall back to the `sha256(path + session_date)` hash using the path and date already stored in the file (existing behavior, unchanged).
      - Caveat: if the transcript is gone by submission time *and* the original format had a native `sessionId`, the fallback hash won't match what grademe stored at grading time — the server may then treat it as a new session instead of a dedup match. Not a failure, just a heads-up if you're pruning transcripts aggressively.
    - `participant` — pass through as-is. The server overrides identity from the API key regardless, so don't rely on this field for auth.

5. **Validate the built payload before sending.** Confirm `/tmp/grademe_submission.json` is syntactically valid JSON before it ever reaches `curl` — this catches any construction bug immediately, instead of it surfacing later as a confusing `400` from the server:
    ```bash
    python3 -m json.tool < /tmp/grademe_submission.json > /dev/null || {
      echo "Payload JSON invalid — aborting, not sending. Re-check step 4 construction."
      exit 1
    }
    ```
    If this fails, stop and report which part looks malformed. Do NOT send a payload already known to be broken, and do NOT try to "patch" it with more manual string edits — re-run Step 4's Python build instead.

6. **POST** via one Bash `curl` (never print the key value). Use `--data-binary` (not `--data`) so the file's bytes are sent exactly as written, with no newline-stripping:
    ```bash
    curl -sS -o /tmp/grademe_upload.json -w '%{http_code}' \
      -X POST "$VIBESCORE_API_URL/scores" \
      -H "Content-Type: application/json" \
      -H "X-API-Key: $VIBESCORE_API_KEY" \
      --data-binary @/tmp/grademe_submission.json
    ```

7. **Report by status** (as returned, don't blindly retry):
   - `201` → "Skor terkirim ke leaderboard." + `participant` + `id` from the response body.
   - `409` → "Sesi ini sudah pernah di-upload (dedup session_id) — skor tidak digandakan."
   - `401` → "VIBESCORE_API_KEY tidak valid/absen — skor TIDAK terkirim."
    - `400` → first check whether Step 5's validation actually ran and passed — if the payload was never JSON-validated before send, that's the bug to fix, not the server. If the payload was valid JSON and the `error` mentions an unknown/unexpected field (e.g. "unknown field", "unexpected field"), strip `session_name`, `compacted`, and `transcript_meta` from the body, resubmit once, and tell the user the backend doesn't yet support the new fields. Otherwise show the `error` field from the response body (payload rejected by contract).
   - anything else / curl failure → report the code + message, don't stay silent.

---

Notes on why the file, not memory:
- Skill invocations (and especially subagents dispatched via Task/Agent) never inherit prior conversation history — they only see what's explicitly written into their prompt.
- Even within the same CLI session, calling `/submit-grade` later gives it a fresh context; nothing from the `/grademe` turn carries over automatically.
- The file is therefore the *only* reliable handoff — which is also why `submit-grade` re-validates it in step 2 instead of trusting it.