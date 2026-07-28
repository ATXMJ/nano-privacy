# 🧠 Agent Memory & Operational Preferences

> [!NOTE]
> This file contains persistent operational learnings, command preferences, and environment-specific notes for AI agents working in this repository.

## 🛠️ Tool & Command Preferences
- **Build Targets**: Builds, tests, and code quality workflows are managed locally within sub-repos via `Justfile` (`nano-node/` and `rsnano-node/`).
- **Testing**: Use `just test` in the respective sub-repository directory.
- **Submodule Initializer**: Run `git submodule update --init --recursive` in `nano-node/` before running CMake build.

## ⚠️ Environment Gotchas & Quirks
- `rsnano-node` lives under `atxmj-rsnano` org on GitHub to avoid fork network collisions.

## ⚙️ Workspace Ecosystem Configuration
- **Issue / Ticket Tracker**: GitHub Issues (`https://github.com/ATXMJ/nano-node/issues`, `https://github.com/atxmj-rsnano/rsnano-node/issues`)
- **Cross-Repo Hub**: GitHub Wiki (`git@github.com:ATXMJ/nano-node.wiki.git`)
- **Integration Tools**: `github` MCP server

## 📝 Active Session Notes (Transient)
- Established workspace memory & cross-repo state pattern using standard Agent Skills (`.agents/skills/workspace-memory/SKILL.md`).
