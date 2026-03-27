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
SPRINT_FILE="$ROOT/.gstack-sprint.json"

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
  GSTACK_BIN="$HOME/.codex/skills/gstack/bin"
  if [ ! -x "$GSTACK_BIN/sprint-setup" ]; then
    GSTACK_BIN="$HOME/.claude/skills/gstack/bin"
  fi
  "$GSTACK_BIN/sprint-setup" "$WS" "$ROOT"
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
   `jq --argjson id <TASK_ID> '(.tasks[] | select(.id == $id)).status = "review"' .gstack-sprint.json > .gstack-sprint.json.tmp && mv .gstack-sprint.json.tmp .gstack-sprint.json`
