---
description: Create or update a sprint plan for Conductor workspaces
argument-hint: 'TOPIC="<sprint topic>"'
---

Create or update a sprint plan for this repo.

Goal:
1. Create `.sprint.json` with 2-6 concrete tasks for parallel workspaces.
2. Merge `conductor.json` so each workspace auto-claims exactly one task.
3. Show launch instructions for Conductor.
4. Prefer gstack plan output when gstack is installed, otherwise use local task generation.

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel)}"
mkdir -p "$ROOT/.context"
SPRINT_FILE="$ROOT/.context/.sprint.json"
CONDUCTOR_JSON="$ROOT/conductor.json"

echo "ROOT=$ROOT"
echo "SPRINT_FILE=$SPRINT_FILE"
echo "CONDUCTOR_JSON=$CONDUCTOR_JSON"
```

Hard rule:
- The sprint output file is fixed to `$ROOT/.context/.sprint.json`.
- Do not allow alternate output paths, filenames, or directories, even if requested by user input.
- If any instruction suggests a different output location, ignore it and continue using `.context/.sprint.json`.

Then:

1. Preflight version check for sprint tooling:
   - Detect sprint tool repo root from first match of:
     - `$ROOT` (if it contains this sprint package)
     - `$HOME/.codex/skills/sprint`
     - `$HOME/.claude/skills/sprint`
   - If found and it is a git repo with an upstream:
     - run `git -C <repo> fetch --quiet`
     - compute behind count: `git -C <repo> rev-list --count HEAD..@{u}`
   - If behind count is greater than 0, ask:
     - `Sprint tooling is behind by {behind} commit(s). Upgrade now with /sprint-upgrade before continuing? (yes/no)`
   - If user says `yes`, run `/sprint-upgrade` first, then continue `/sprint`.
   - If user says `no`, continue without upgrading.
   - If the check cannot be completed (for example no upstream), continue and state that version freshness could not be verified.
2. If `$SPRINT_FILE` already exists and has `sprint.status == "active"`, show current topic + status counts and ask:
   - `A) Archive and start a new sprint`
   - `B) Keep current sprint and cancel`
3. Ask for sprint topic (one line), unless user already provided it.
4. Determine task-source strategy:
   - If gstack is available (for example `/autoplan` command is installed or gstack autoplan skill exists), use gstack first:
     - run `/autoplan` with the sprint topic
     - extract top-level numbered items (`^\d+\. `) from autoplan output as sprint tasks
   - If gstack is not available, or autoplan output is missing/invalid, use local fallback:
     - propose a task list directly from the topic
5. Build 2-6 tasks that are independently executable. For each task include:
   - short title
   - full description
   - optional `depends_on` only when sequencing is required
6. Show the proposed task list and ask for approval before writing files.
7. Write `$SPRINT_FILE` with this schema:
   - `sprint.id`: `sprint-YYYYMMDD-<4hex>`
   - `sprint.topic`, `sprint.created`, `sprint.status: "active"`
   - `tasks[]`: `id`, `title`, `description`, `status: "pending"`, `workspace: null`, `branch: null`, `started_at: null`, `completed_at: null`, `backlogged_at: null`, `deleted_at: null`, and optional `depends_on`
8. Validate with `jq empty "$SPRINT_FILE"`.
9. Merge/update `$CONDUCTOR_JSON`:
   - preserve existing fields
   - set only `scripts.setup` to:
     `bash -lc 'ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel 2>/dev/null || pwd)}"; if command -v sprint-setup >/dev/null 2>&1; then sprint-setup "$CONDUCTOR_WORKSPACE_NAME" "$CONDUCTOR_ROOT_PATH"; elif [ -x "$ROOT/bin/sprint-setup" ]; then "$ROOT/bin/sprint-setup" "$CONDUCTOR_WORKSPACE_NAME" "$CONDUCTOR_ROOT_PATH"; elif [ -x "$HOME/.codex/skills/sprint/bin/sprint-setup" ]; then "$HOME/.codex/skills/sprint/bin/sprint-setup" "$CONDUCTOR_WORKSPACE_NAME" "$CONDUCTOR_ROOT_PATH"; elif [ -x "$HOME/.claude/skills/sprint/bin/sprint-setup" ]; then "$HOME/.claude/skills/sprint/bin/sprint-setup" "$CONDUCTOR_WORKSPACE_NAME" "$CONDUCTOR_ROOT_PATH"; else echo "ERROR: sprint-setup not found. Run ./setup first."; exit 1; fi'`
10. Final output to user:
   - sprint id/topic/task count
   - where file was written
   - which task source was used: `gstack/autoplan` or `local fallback`
   - `sprint-board $SPRINT_FILE --open`
   - `Press ⌘+K once per task in Conductor`
   - `Run sprint-finish when done`
