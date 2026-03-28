---
name: sprint-board
description: Generate an openable HTML sprint board from .sprint.json with progress and dependency status.
---

Generate an HTML sprint status board users can open in a browser.

Sprint file location rules:
- First: `$CONDUCTOR_ROOT_PATH/.context/.sprint.json` (if `CONDUCTOR_ROOT_PATH` is set)
- Fallback: `.context/.sprint.json` in the current directory

Output behavior:
- Write HTML to `.context/sprint-board.html` (or sibling directory of the sprint file)
- Include a task table with: task id, title, status, assigned workspace, branch, and elapsed time since `started_at` (or `not started`)
- Include a task dependency graph based on `depends_on` (not a dependency table), visually marking `resolved` vs `blocked` edges
- Include sprint-level progress counts: done/total, backlog, deleted, review, in-progress, pending
- Return the absolute output path and a one-line open command (`open <path>` on macOS)

If no sprint file is found, say exactly:
`No active sprint found. Run /sprint to start one.`
