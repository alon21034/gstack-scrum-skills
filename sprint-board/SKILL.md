---
name: sprint-board
description: Generate sprint board output from .sprint.json (HTML or text) with progress and dependency status.
---

Generate sprint status output from `.sprint.json`.

Sprint file location rules:
- First: `$CONDUCTOR_ROOT_PATH/.context/.sprint.json` (if `CONDUCTOR_ROOT_PATH` is set)
- Fallback: `.context/.sprint.json` in the current directory

Output behavior:
- Default mode writes HTML to `.context/sprint-board.html` (or sibling directory of the sprint file) and opens it automatically.
- `--no-open` writes HTML without opening a browser.
- `--text` prints a text-only board to stdout (no HTML output file).
- Include a task table with: task id, title, status, assigned workspace, branch, and elapsed time since `started_at` (or `not started`)
- Include a task dependency graph based on `depends_on` (not a dependency table), visually marking `resolved` vs `blocked` edges
- Include sprint-level progress counts: done/total, backlog, deleted, review, in-progress, pending
- In HTML mode, return the absolute output path.

If no sprint file is found, say exactly:
`No active sprint found. Run /sprint to start one.`
