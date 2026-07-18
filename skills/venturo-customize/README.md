# Venturo Customize Skills

This skill set provides two Opencode skills for grading and submitting vibe-coding practice sessions to a leaderboard:

1. **grademe** - Grade the user's vibe-coding practice from a session transcript against a 7-dimension rubric (total 100 points). Outputs JSON (contract with vibescore-api) and a short narrative in the requested language.

2. **submit-grade** - Upload graded results to the vibescore leaderboard. Reads from `~/.vibescore/grades/` and submits via API. 

> **Note for non-Venturo users:** This skill is designed for the Venturo vibescore leaderboard. If you don't have Venturo credentials, you can still use `grademe` for local grading, but `submit-grade` will only run in dry-run mode. To skip submission entirely, simply don't configure the API credentials.


## Why This Exists

This skill set is derived from the [venturo-claude](https://github.com/venturo-id/venturo-claude) project, which provides similar functionality for Claude Code. The venturo-customize skills have been adapted for use with others Agentic AI.

## How These Skills Work

### grademe
- Locates session transcripts from various sources (Opencode, Claude Code, Kiro, Freebuff)
- Dispatches a grading subagent that evaluates the transcript against a rubric
- Returns JSON results and a narrative summary
- Persists graded results to `~/.vibescore/grades/` for later submission

### submit-grade
- Reads persisted grade files from `~/.vibescore/grades/`
- Validates the grade data
- Submits to the vibescore leaderboard API
- Handles deduplication and error reporting

## Getting Started

Users cannot install these skills via `npx` or `npm`. To use these skills:

1. **Sync the skills to your agent using the subcommands skill** (Recommended):
   ```
   /subcommands
   ```
   This will detect your agent (Kiro, OpenCode, Freebuff, or other) and create the appropriate native commands pointing to the skill files in `~/.ai/skills/`.
   
   > **Note:** You must install the subcommands skill first. See [subcommands/README.md](../subcommands/README.md) for installation and usage.

2. Place the skill files in your skills directory:
   ```bash
   cp /path/to/vibe-playground/skills/venturo-customize/grademe.md ~/.ai/skills/
   cp /path/to/vibe-playground/skills/venturo-customize/submit-grade.md ~/.ai/skills/
   ```

3. **Guide your agent to use the skills**:
   > "Grade my current session using the grademe skill"
   
   Or with options:
   > "Grade my session at /path/to/session.jsonl in English"
   
   After grading, submit to the leaderboard:
   > "Submit my latest grade to the vibescore leaderboard using submit-grade"

## Usage

After syncing, invoke the skills with:
```bash
# Grade a session (defaults to Indonesian language, use locale=en for English)
# Required: participant name as first argument
/grademe <participant-name>
# or with transcript path
/grademe <participant-name> path=/path/to/session.jsonl
# or specify language
/grademe <participant-name> locale=en

# Submit the most recent grade
/submit-grade
# or specify a particular grade file
/submit-grade /path/to/grade-file.json
```

## Original Reference

The original implementation and documentation can be found at:
https://github.com/venturo-id/venturo-claude

For detailed information about the grading rubric, workflow, and implementation details, refer to the individual skill READMEs in the venturo-claude repository.