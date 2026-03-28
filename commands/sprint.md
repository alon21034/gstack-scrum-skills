---
description: Create or update a sprint plan with tasks and acceptance criteria
argument-hint: 'TOPIC="<sprint topic>"'
---

Create or update a sprint plan for this repo.

Goal:
1. Create `.sprint.json` with 2-6 concrete tasks, each with acceptance criteria.
2. Show task summary and next steps.
3. Prefer design docs in `.context/`, then gstack plan output, otherwise use local task generation.

Run this bash first:

```bash
set -euo pipefail
PATH="/opt/homebrew/bin:/usr/local/bin:$PATH"

ROOT="${CONDUCTOR_ROOT_PATH:-$(git rev-parse --show-toplevel)}"
mkdir -p "$ROOT/.context"
SPRINT_FILE="$ROOT/.context/.sprint.json"

echo "ROOT=$ROOT"
echo "SPRINT_FILE=$SPRINT_FILE"
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
4. Determine task-source strategy (in priority order):
   - **Design documents:** Check `.context/` for `*.md` files (excluding sprint files). If found, list them and ask the user whether to use them as task source. If yes, read the design docs and decompose into tasks.
   - **Autoplan:** If no design docs (or user declines), and gstack is available, run `/autoplan` with the sprint topic and extract top-level numbered items.
   - **Local fallback:** If neither is available, propose a task list directly from the topic.
5. Build 2-6 tasks that are independently executable. For each task include:
   - short title
   - full description with `## Acceptance Criteria` section at the end
   - acceptance criteria: 2-5 concrete, verifiable checklist items per task (e.g., "- [ ] API returns 200 for valid input"). Derive from design doc requirements or task description.
   - optional `depends_on` only when sequencing is required
6. Show the proposed task list and ask for approval before writing files.
7. Write `$SPRINT_FILE` with this schema:
   - `sprint.id`: `sprint-YYYYMMDD-<4hex>`
   - `sprint.topic`, `sprint.created`, `sprint.status: "active"`
   - `tasks[]`: `id`, `title`, `description` (with `## Acceptance Criteria` section), `status: "pending"`, `started_at: null`, `completed_at: null`, and optional `depends_on`
8. Validate with `jq empty "$SPRINT_FILE"`.
9. Final output to user:
   - sprint id/topic/task count
   - where file was written
   - which task source was used: `design docs`, `gstack/autoplan`, or `local fallback`
   - `Pick a task and run /sprint-task to start`
   - `View progress: sprint-board $SPRINT_FILE`
   - `Run sprint-finish when done`
