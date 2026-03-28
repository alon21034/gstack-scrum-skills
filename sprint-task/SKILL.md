---
name: sprint-task
description: Pick a sprint task, implement it, and verify acceptance criteria before marking review.
---

Manual sprint task entrypoint.

Goal:
1. Show available tasks and let the user pick one (or use a provided task ID).
2. Show full task details including acceptance criteria.
3. Ask user approval before implementing.
4. After implementation, verify acceptance criteria before marking review.

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel)}"
SPRINT_FILE="$ROOT/.context/.sprint.json"

if [ ! -f "$SPRINT_FILE" ]; then
  echo "ERROR: No sprint file found at $SPRINT_FILE. Run /sprint first."
  exit 1
fi

STATUS="$(jq -r '.sprint.status' "$SPRINT_FILE")"
if [ "$STATUS" != "active" ]; then
  echo "ERROR: Sprint is not active (status=$STATUS)."
  exit 1
fi

echo "SPRINT_TOPIC=$(jq -r '.sprint.topic' "$SPRINT_FILE")"
echo "PENDING_TASKS:"
jq -r '.tasks[] | select(.status == "pending") | "  \(.id). \(.title)"' "$SPRINT_FILE"
echo "IN_PROGRESS_TASKS:"
jq -r '.tasks[] | select(.status == "in-progress") | "  \(.id). \(.title)"' "$SPRINT_FILE"
```

Then:

1. If a task ID was provided, use that task. Otherwise, show pending tasks and ask the user to pick one.
2. Read the selected task's full description:
   `jq -r --argjson id <TASK_ID> '.tasks[] | select(.id == $id) | .description' "$SPRINT_FILE"`
3. Show the user:
   - Sprint topic
   - Task id and title
   - Full task description (including acceptance criteria)
   - Current git branch
4. Ask:
   "Approve start implementation for this task? (yes/no)"
5. Do not start implementation until user explicitly approves.
6. If approved, mark task as in-progress:
   `jq --argjson id <TASK_ID> '(.tasks[] | select(.id == $id)).status = "in-progress" | (.tasks[] | select(.id == $id)).started_at = "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"' "$ROOT/.context/.sprint.json" > "$ROOT/.context/.sprint.json.tmp" && mv "$ROOT/.context/.sprint.json.tmp" "$ROOT/.context/.sprint.json"`
7. Implement only this task's scope.
8. When implementation is complete, verify acceptance criteria before marking review:
   - Parse the `## Acceptance Criteria` section from the task description
   - Check each criterion against the actual implementation
   - Report results in this format:
     ```
     Acceptance Criteria Check:
     [PASS] {criterion 1}
     [PASS] {criterion 2}
     [FAIL] {criterion 3} — {reason}
     Result: X/Y passed
     ```
   - If ALL criteria pass: proceed to mark as "review"
   - If any criteria FAIL: show the report and ask user:
     "Some acceptance criteria failed. Options: A) Fix the failing criteria B) Mark as review anyway C) Backlog this task"
9. When all criteria pass (or user approves despite failures), set task to review:
   `jq --argjson id <TASK_ID> '(.tasks[] | select(.id == $id)).status = "review"' "$ROOT/.context/.sprint.json" > "$ROOT/.context/.sprint.json.tmp" && mv "$ROOT/.context/.sprint.json.tmp" "$ROOT/.context/.sprint.json" && (sprint-board "$ROOT/.context/.sprint.json" >/dev/null 2>&1 || "$ROOT/bin/sprint-board" "$ROOT/.context/.sprint.json" >/dev/null 2>&1)`
10. Summarize what was implemented and the acceptance criteria results.
