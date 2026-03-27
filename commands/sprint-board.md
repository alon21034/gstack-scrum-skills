Read .gstack-sprint.json in the current directory (or $CONDUCTOR_ROOT_PATH/.gstack-sprint.json if set) and render a snapshot of the sprint board as a formatted table.

Show for each task: task id, title, status, assigned workspace, branch, and time elapsed since started_at (or "not started" if null).

Format the output as:

```
═══════════════════════════════════════════════════════════════
  SPRINT: {topic}  [STATUS]  {N} tasks
═══════════════════════════════════════════════════════════════
  #   TITLE                       STATUS          WORKSPACE
───────────────────────────────────────────────────────────────
  1   Auth middleware              IN PROGRESS     workspace-1
  2   Login UI                     IN PROGRESS     workspace-2
  3   Session management           PENDING         —
═══════════════════════════════════════════════════════════════
  Progress: {done}/{total} done   {backlog} backlog   {deleted} deleted   {review} in review   {inprogress} in progress
═══════════════════════════════════════════════════════════════
```

Also show any tasks with depends_on fields and whether their dependencies are resolved.

If no sprint file is found, say: "No active sprint found. Run /sprint to start one."
