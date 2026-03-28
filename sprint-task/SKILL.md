---
name: sprint-task
description: Claim/show one workspace sprint task with approval gate, then implement only after explicit user approval.
---

Manual sprint task entrypoint.

Goal:
1. Ensure this workspace has exactly one in-progress task (claim one if needed).
2. Show full task details to the user.
3. Ask user approval before implementing.

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel)}"
WS="${CONDUCTOR_WORKSPACE_NAME:-$(basename "$PWD")}"
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

TASK_ID="$(jq -r --arg ws "$WS" \
  '.tasks[] | select(.workspace == $ws and .status == "in-progress") | .id' \
  "$SPRINT_FILE" | head -n1)"

if [ -z "${TASK_ID:-}" ] || [ "$TASK_ID" = "null" ]; then
  if command -v sprint-setup >/dev/null 2>&1; then
    sprint-setup "$WS" "$ROOT"
  elif [ -x "$ROOT/bin/sprint-setup" ]; then
    "$ROOT/bin/sprint-setup" "$WS" "$ROOT"
  elif [ -x "$HOME/.codex/skills/sprint/bin/sprint-setup" ]; then
    "$HOME/.codex/skills/sprint/bin/sprint-setup" "$WS" "$ROOT"
  elif [ -x "$HOME/.claude/skills/sprint/bin/sprint-setup" ]; then
    "$HOME/.claude/skills/sprint/bin/sprint-setup" "$WS" "$ROOT"
  else
    echo "ERROR: sprint-setup not found (PATH, repo bin, or ~/.codex|~/.claude skills/sprint)." >&2
    exit 1
  fi
  TASK_ID="$(jq -r --arg ws "$WS" \
    '.tasks[] | select(.workspace == $ws and .status == "in-progress") | .id' \
    "$SPRINT_FILE" | head -n1)"
fi

if [ -z "${TASK_ID:-}" ] || [ "$TASK_ID" = "null" ]; then
  echo "ERROR: No task assigned to workspace $WS."
  exit 1
fi

echo "SPRINT_TOPIC=$(jq -r '.sprint.topic' "$SPRINT_FILE")"
echo "TASK_ID=$TASK_ID"
echo "TASK_TITLE=$(jq -r --argjson id "$TASK_ID" '.tasks[] | select(.id == $id) | .title' "$SPRINT_FILE")"
echo "TASK_DESC<<__TASK_DESC__"
jq -r --argjson id "$TASK_ID" '.tasks[] | select(.id == $id) | .description' "$SPRINT_FILE"
echo "__TASK_DESC__"
```

Then:

1. Show the user:
   - Sprint topic
   - Task id and title
   - Full task description
   - Current git branch
2. Ask:
   "Approve start implementation for this task? (yes/no)"
3. Do not start implementation until user explicitly approves.
4. If approved, implement only this task scope.
5. When done, set task to review:
   `jq --argjson id <TASK_ID> '(.tasks[] | select(.id == $id)).status = "review"' "$ROOT/.context/.sprint.json" > "$ROOT/.context/.sprint.json.tmp" && mv "$ROOT/.context/.sprint.json.tmp" "$ROOT/.context/.sprint.json" && (sprint-board "$ROOT/.context/.sprint.json" >/dev/null 2>&1 || "$ROOT/bin/sprint-board" "$ROOT/.context/.sprint.json" >/dev/null 2>&1)`
6. If pending tasks are blocked by `review` dependencies, rely on `sprint-setup`'s git-log check (on `main`) to auto-mark already-merged review tasks as `done` so they stop blocking.
