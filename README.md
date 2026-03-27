# claude-skills

Personal Claude Code skills and tools.

## /sprint — Multi-Agent Sprint Coordinator

Lightweight Scrum coordination layer for parallel AI development with [Conductor](https://docs.conductor.build/).

**The idea:** type a sprint topic, get a sprint board, press ⌘+K in Conductor — each workspace auto-claims a task and starts executing. You only think about what to build next.

### Files

| File | Purpose |
|------|---------|
| `sprint/SKILL.md.tmpl` | Skill source (edit this) |
| `sprint/SKILL.md` | Generated skill (don't edit) |
| `bin/sprint-setup` | Conductor workspace bootstrap — claims task, creates branch |
| `bin/sprint-board` | Live terminal board (`watch -n 3` + `jq`) |
| `bin/sprint-approve` | Mark task done after review |
| `commands/sprint-board.md` | `/sprint-board` slash command (one-shot snapshot) |
| `commands/sprint-approve.md` | `/sprint-approve` slash command |

### Install

```bash
# Copy skill to gstack
cp sprint/SKILL.md ~/.claude/skills/gstack/sprint/SKILL.md
cp sprint/SKILL.md.tmpl ~/.claude/skills/gstack/sprint/SKILL.md.tmpl

# Copy scripts
cp bin/sprint-setup ~/.claude/skills/gstack/bin/sprint-setup
cp bin/sprint-board ~/.claude/skills/gstack/bin/sprint-board
cp bin/sprint-approve ~/.claude/skills/gstack/bin/sprint-approve
chmod +x ~/.claude/skills/gstack/bin/sprint-{setup,board,approve}

# Copy slash commands
cp commands/sprint-board.md ~/.claude/commands/sprint-board.md
cp commands/sprint-approve.md ~/.claude/commands/sprint-approve.md
```

### Prerequisites

- [Conductor](https://docs.conductor.build/) (Mac app)
- `jq` — `brew install jq`
- `watch` — `brew install watch`

### Usage

1. In any CC session in your project: `/sprint`
2. Describe your sprint topic (or use an existing `/autoplan` output)
3. Open Conductor, press ⌘+K once per task
4. Each workspace auto-starts executing its task
5. Watch progress: `sprint-board .gstack-sprint.json`
6. When a task shows REVIEW: `sprint-approve <task-id>`
