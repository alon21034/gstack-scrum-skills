Mark the specified task as done. The user will provide a task ID (e.g. "/sprint-approve 2").

Steps:
1. Parse the task ID from the user's input (the number after /sprint-approve).
2. Find the sprint file: check $CONDUCTOR_ROOT_PATH/.gstack-sprint.json, then .gstack-sprint.json in the current directory.
3. Validate the task exists and is in "review" status. If not in "review", say: "Task {id} is in '{status}' status, not 'review'. The agent must mark it as review before you can approve."
4. Update the task: set status = "done" and completed_at = current ISO timestamp.
5. Print: "Task {id} done: {title}. Sprint progress: {done}/{total} complete."
6. If all tasks are done, also set sprint.status = "complete" and print: "Sprint complete: {topic}"

If no task ID is provided, list all tasks in "review" status and ask which one to approve.
If the sprint file is not found, say: "No sprint file found. Run /sprint to start a sprint."
