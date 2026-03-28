# Changelog

All notable changes to this project will be documented in this file.

## 0.3.0 - 2026-03-29

### Added

- Design doc sourcing and acceptance criteria support (#20).

### Changed

- Removed workspace coupling from sprint configuration (#20).

## 0.2.0 - 2026-03-28

### Added

- HTML sprint board with dependency graph rendering and auto-refresh (#16).
- `sprint-upgrade` command for self-upgrading the sprint skills package (#14).
- Upgrade checks: sprint commands now warn when a newer version is available (#18).
- POSIX-compatible `mkdir`-based file locking, replacing `flock` for broader compatibility (#14).

### Improved

- Sprint blocker and board UX: clearer status indicators and interaction flow (#17).
- Pending/blocked tasks now prompt the user with a 20-minute retry window and timeout messaging (#14).

### Fixed

- Blocked sprint tasks are correctly kept in pending state instead of being skipped (#15).
- Sprint file (`.sprint.json`) now written to `.context/` shared directory so Conductor workspaces can read it (#13).
- Installer copies `sprint-board` and `sprint-finish` commands into each workspace (#13).

## 0.1.3 - 2026-03-28

### Fixed

- Added top-level skill alias links (`sprint-task`, `sprint-board`, `sprint-finish`) during install so Claude/Conductor environments that scan one level of `~/.{codex|claude}/skills` can discover them.

## 0.1.2 - 2026-03-28

### Added

- Added standalone Conductor-visible skills: `sprint-task`, `sprint-board`, `sprint-finish`.
- Installer now deploys those skill manifests under the sprint package install root.

## 0.1.1 - 2026-03-28

### Fixed

- Codex install target now uses `~/.codex/prompts` for slash commands instead of `~/.codex/commands`.
- Added YAML frontmatter to `commands/sprint*.md` files so Codex recognizes them as custom prompts.
- Updated README verification examples to use the Codex prompts directory.

## 0.1.0 - 2026-03-28

Initial release.

### Added

- `sprint` skill package for multi-agent sprint coordination in Conductor.
- Helper scripts: `sprint-setup`, `sprint-board`, and `sprint-finish`.
- Slash command docs: `/sprint`, `/sprint-task`, `/sprint-board`, and `/sprint-finish`.
- Host-aware installer support for Codex and Claude install paths.
