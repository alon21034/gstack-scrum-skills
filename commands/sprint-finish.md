Finalize the active sprint by resolving all non-terminal tasks.

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel)}"

if command -v sprint-finish >/dev/null 2>&1; then
  sprint-finish
elif [ -x "$ROOT/bin/sprint-finish" ]; then
  "$ROOT/bin/sprint-finish"
elif [ -x "$HOME/.codex/skills/sprint/bin/sprint-finish" ]; then
  "$HOME/.codex/skills/sprint/bin/sprint-finish"
elif [ -x "$HOME/.claude/skills/sprint/bin/sprint-finish" ]; then
  "$HOME/.claude/skills/sprint/bin/sprint-finish"
else
  echo "ERROR: sprint-finish not found. Run ./setup for this repo first."
  exit 1
fi
```

Behavior requirements:
1. Resolve every task not in `done`, `backlog`, or `deleted`.
2. For each unresolved task, prompt for one action: `done`, `backlog`, `deleted`.
3. If marking a `pending` or `in-progress` task as `done`, show warning and require explicit yes/no confirmation.
4. If any unresolved tasks remain (for example, user aborts), leave sprint active and print unresolved list.
5. When all tasks are terminal (`done/backlog/deleted`), set `sprint.status = "complete"`, set `sprint.closed_at` if missing, and print final counts.
