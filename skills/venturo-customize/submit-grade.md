---
name: submit-grade
description: Upload graded session to vibescore leaderboard. Requires API credentials.
---

# submit-grade — Leaderboard Uploader

Standalone companion to `grademe`. Reads **newest** file from `~/.vibescore/grades/` and uploads to vibescore leaderboard.

**No arguments, paths, or flags** — always processes only the most recent graded file. Does NOT regrade, NOT re-check evidence, has NO access to prior conversation. Trusts only the JSON file on disk (re-validated, never assumed correct).

**One exception:** independently re-derives `session_id` (Step 4) — touches transcript path only, never content/evidence.

## Precondition

Requires file at `~/.vibescore/grades/{GRADED_AT}_{SESSION_ID}.json` (produced by grademe's Persist step). Timestamp-led filename → "newest" = plain lexical sort (no mtime needed). `schema_version` `"1.1"`, includes `grade` + `transcript_meta`.

No file? Tell user to run `/grademe` first. Do NOT regrade, summarize, or guess.

## Workflow

1. **Resolve source file**
    - Newest file in `~/.vibescore/grades/`. Lexical sort (not `ls -t` — breaks on sync/copy):
      ```bash
      ls ~/.vibescore/grades/*.json 2>/dev/null | sort | tail -1
      ```
    - Nothing found → tell user to run `/grademe` first. No guess/regrade.

2. **Load & re-validate** (never trust blindly — may be hand-edited or old version)
    - `schema_version`: accepts `"1.0"` or `"1.1"`. `"1.1"` requires `session_name`/`compacted`/`transcript_meta`.
    - `grade` contract: `participant`, `session_date`, `total_score`, `breakdown`, `misses`, `next_session_advice`. `prompt_analysis` optional.
    - Breakdown maxes (by field name): planning 20 / context 20 / decomposition 15 / delegation 15 / verification 15 / token_efficiency 10 / documentation 5. Sum = `total_score`.
    - `misses` non-empty → `total_score` ≤ 94. `misses` empty → `total_score` ≥ 95.
    - Any check fails → report exactly what failed. Do NOT silently fix numbers.

3. **Check credentials**
    - Both `VIBESCORE_API_URL` + `VIBESCORE_API_KEY` set → proceed to step 4.
    - Either missing → show payload that *would* be sent, skip POST (not error). Tell user: generate token at `https://vibescore.venturo.pro/participants/`, export both env vars (example URL: `https://vibescore-be.venturo.pro`). Do NOT invent URL or key.

4. **Build submission body — programmatically, never by hand**
    `misses`, `next_session_advice`, and `session_name` are free text (non-English, quotes, backslashes, citations). Re-typing into bash strings/heredocs breaks escaping — always use JSON serializer from parsed source.

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

    - Copy `grade.*`, strip `prompt_analysis` (not in leaderboard contract; show separately). `session_name`/`compacted`/`transcript_meta` forwarded as-is (grademe is source of truth).
    - `session_id` — **recompute independently**, same tiered logic as grademe:
      1. Native ID: Claude Code → `sessionId` field; opencode → `s.id`; Kiro CLI → filename; Freebuff → timestamp folder.
      2. Tier 1 unavailable → filename stem (`basename`).
      3. Tier 2 generic (e.g. `"log"`) → `sha256(transcript_path + session_date)` first 16 hex chars.
      - Transcript gone? Fall back to hash from stored file.
      - Caveat: native `sessionId` exists but transcript gone → hash ≠ grademe's. Server may treat as new session. Not failure.
    - `participant` — pass through. Server overrides from API key, don't rely for auth.

5. **Validate payload before sending** — catch construction bugs before `curl`:
    ```bash
    python3 -m json.tool < /tmp/grademe_submission.json > /dev/null || {
      echo "Payload JSON invalid — aborting, not sending. Re-run Step 4 ."
      exit 1
    }
    ```
    Fail → report, do NOT send broken payload or patch with manual edits. Re-run Step 4 Python build.

6. **POST** — one `curl` call (never print key). `--data-binary` (not `--data`) preserves bytes exactly:
    ```bash
    curl -sS -o /tmp/grademe_upload.json -w '%{http_code}' \
      -X POST "$VIBESCORE_API_URL/scores" \
      -H "Content-Type: application/json" \
      -H "X-API-Key: $VIBESCORE_API_KEY" \
      --data-binary @/tmp/grademe_submission.json
    ```

7. **Report by status** (don't blindly retry):
   - `201` → "Skor terkirim ke leaderboard." + `participant` + `id` from response.
   - `409` → Sesi sudah di-upload (dedup) — skor tidak digandakan.
   - `401` → API key invalid/missing — skor TIDAK terkirim.
   - `400` → Check Step 5 validation ran. If valid JSON and error mentions unknown/unexpected field → strip `session_name`/`compacted`/`transcript_meta`, resubmit once, tell user backend doesn't support new fields yet. Otherwise show `error` field.
   - Other failure → report code + message.

---

**Why file, not memory:** Skill invocations (especially subagents) never inherit conversation history. Each `/submit-grade` call is fresh context. File is the *only* reliable handoff — hence re-validation in step 2.