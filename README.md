# claude-skills

Personal Claude Code and Codex skills and tools.

## /sprint — Multi-Agent Sprint Coordinator

Lightweight Scrum coordination layer for parallel AI development with [Conductor](https://docs.conductor.build/).

**The idea:** type a sprint topic, get a sprint board, press ⌘+K in Conductor — each workspace auto-claims a task and starts executing. You only think about what to build next.

### Files

| File | Purpose |
|------|---------|
| `setup` | One-command installer (`./setup`) for skill + scripts + slash commands |
| `sprint/SKILL.md.tmpl` | Skill source (edit this) |
| `sprint/SKILL.md` | Generated skill (don't edit) |
| `bin/sprint-setup` | Conductor workspace bootstrap — claims task, creates branch |
| `bin/sprint-board` | Live terminal board (`watch -n 3` + `jq`) |
| `bin/sprint-approve` | Mark task done after review |
| `commands/sprint.md` | `/sprint` slash command (entry point) |
| `commands/sprint-task.md` | `/sprint-task` slash command (manual claim + approval gate) |
| `commands/sprint-board.md` | `/sprint-board` slash command (one-shot snapshot) |
| `commands/sprint-approve.md` | `/sprint-approve` slash command |

### Install

This now follows the same pattern as upstream gstack: `git clone` + `./setup`.

1. Install gstack first (if not already installed):

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.codex/skills/gstack
cd ~/.codex/skills/gstack && ./setup
```

2. Install this sprint package:

```bash
git clone --single-branch --depth 1 https://github.com/alon21034/gstack-scrum-skills.git ~/.codex/skills/gstack-sprint
cd ~/.codex/skills/gstack-sprint && ./setup
```

3. Verify files landed in the expected locations:

```bash
ls ~/.codex/skills/gstack/bin/sprint-{setup,board,approve}
ls ~/.codex/skills/gstack/sprint/SKILL.md
ls ~/.codex/commands/sprint-{task,board,approve}.md
```

Optional: use custom install paths (for repo-local gstack installs):

```bash
./setup --gstack-root .codex/skills/gstack --commands-dir .codex/commands
```

If you use Claude Code paths, replace `.codex` with `.claude` in the commands above.

### Prerequisites

- [gstack](https://github.com/garrytan/gstack) installed and set up (`./setup`)
- [Conductor](https://docs.conductor.build/) (Mac app)
- `jq` — `brew install jq`
- `watch` — `brew install watch`

### Usage

1. In any CC session in your project: `/sprint`
2. Describe your sprint topic (or use an existing `/autoplan` output)
3. Open Conductor, press ⌘+K once per task
4. In each workspace, run `/sprint-task` to claim/view a task, then approve before implementation starts
5. Watch progress: `sprint-board .gstack-sprint.json`
6. When a task shows REVIEW: `sprint-approve <task-id>`
