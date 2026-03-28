---
name: sprint-upgrade
description: Upgrade sprint skills package and reinstall commands.
---

Upgrade the sprint skills package to the latest revision and reinstall command/skill files.

Tip: check-only mode is available via `sprint-upgrade --check` (add `--quiet` to suppress non-actionable output).

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel 2>/dev/null || pwd)}"

if command -v sprint-upgrade >/dev/null 2>&1; then
  sprint-upgrade
elif [ -x "$ROOT/bin/sprint-upgrade" ]; then
  "$ROOT/bin/sprint-upgrade"
elif [ -x "$HOME/.codex/skills/sprint/bin/sprint-upgrade" ]; then
  "$HOME/.codex/skills/sprint/bin/sprint-upgrade"
elif [ -x "$HOME/.claude/skills/sprint/bin/sprint-upgrade" ]; then
  "$HOME/.claude/skills/sprint/bin/sprint-upgrade"
else
  echo "ERROR: sprint-upgrade not found. Install sprint skills first, then run ./setup."
  exit 1
fi
```

Behavior requirements:
1. Detect the sprint repository root (or use an explicit `--repo` argument if provided).
2. Refuse to run if the target repository has uncommitted changes.
3. Upgrade with fast-forward-only git operations (no merge commits).
4. Re-run `./setup` after update so installed commands and skills are refreshed.
5. Print what changed (or that it was already up to date).
6. Support check-only mode:
   - `sprint-upgrade --check` exits `0` if up to date, `10` if upgrade available, `11` if freshness cannot be verified.
