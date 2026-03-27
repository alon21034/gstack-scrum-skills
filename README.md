# gstack-scrum-skills

Personal Claude Code and Codex skills and tools.

## Compatibility

Install paths are deterministic and host-based.

## /sprint — Multi-Agent Sprint Coordinator

Lightweight Scrum coordination layer for parallel AI development with [Conductor](https://docs.conductor.build/).

**The idea:** type a sprint topic, get a sprint board, press ⌘+K in Conductor, and each workspace can auto-claim a task and start executing. You only think about what to build next.

### Purpose

`/sprint` exists to make parallel AI execution predictable instead of chaotic. It gives one shared sprint file and task lifecycle that every workspace follows.

- Break one feature goal into explicit, independently executable tasks.
- Auto-assign clear ownership so each workspace knows what to do next.
- Keep humans in control with an explicit approval gate before implementation.
- Expose live progress and blockers in a single terminal board.
- Close the sprint cleanly by resolving leftover tasks and preserving state.

### Skill Chart

| Skill/Command | Purpose | Typical Use |
|---|---|---|
| `/sprint` | Create sprint plan and write `.sprint.json` | Start a new parallel sprint |
| `/sprint-task` | Claim/show one workspace task with approval gate | Manual task claiming or task review |
| `/sprint-board` | Snapshot sprint progress table | Quick status check across workspaces |
| `/sprint-finish` | Resolve remaining tasks and close sprint | End sprint and finalize statuses |

### Usage

1. In any CC session in your project: `/sprint`
2. Describe your sprint topic (or provide an existing plan/task list)
3. Task generation behavior:
   - If gstack `/autoplan` is available, sprint tasks are derived from that output.
   - If gstack is not available, tasks are generated locally from your topic.
4. Choose task-claim mode:
   - Auto-claim (recommended): run this repo's `./setup` first, then make sure `conductor.json` has `scripts.setup` wired to `sprint-setup` (the `/sprint` command writes this). After that, press ⌘+K once per task and each workspace auto-claims.
   - Manual claim: if `scripts.setup` is not configured for `sprint-setup`, open each workspace and run `/sprint-task` to claim/view a task.
5. Implement after explicit approval
6. Watch progress: `sprint-board .sprint.json`
7. When implementation/review cycle is done, run: `sprint-finish`

### Prerequisites

- [Conductor](https://docs.conductor.build/) (Mac app)
- [gstack](https://github.com/garrytan/gstack) (optional, used for `/autoplan`-based task generation)
- `jq` — `brew install jq`
- `flock` — `brew install flock`
- `watch` — `brew install watch`

### Install

1. Install this sprint package:

```bash
git clone --single-branch --depth 1 https://github.com/alon21034/gstack-scrum-skills.git ~/.codex/skills/sprint
cd ~/.codex/skills/sprint && ./setup
```

2. Choose host explicitly when needed:

```bash
./setup --host codex
./setup --host claude
./setup --host auto   # installs to both if both are present
```

3. Verify files landed in the expected locations:

```bash
ls ~/.codex/commands/sprint{,-task,-board,-finish}.md
ls ~/.codex/skills/sprint/bin/sprint-{setup,board,finish}
ls ~/.codex/skills/sprint/sprint/SKILL.md
```

Optional: use custom install paths:

```bash
./setup --skill-root .codex/skills/sprint --commands-dir .codex/commands
```
