# ⚙️ Agent Meta-Directives & Initialization Protocol

This workspace implements a modular agent architecture built on [AGENTS.md](https://github.com/agentsmd/agents.md) and [Agent Skills](https://github.com/agentskills/agentskills).

## 🛠️ Master Workspace Skills

All executable workflows and protocols are encapsulated in `.agents/skills/`:

1. **`agent-bootstrap`** ([`SKILL.md`](file://.agents/skills/agent-bootstrap/SKILL.md)):
   - Auto-wires `.agents/skills/` to the native agent runtime environment (preferring references/proxy files over symlinks).
   - Enforces Dual-Documentation (`README.md` for human context + `AGENTS.md` for agent execution) across all modules.

2. **`workspace-memory`** ([`SKILL.md`](file://.agents/skills/workspace-memory/SKILL.md)):
   - Manages local agent operational preferences (`MEMORY.md`) and Architectural Decision Records (`DECISIONS.md`).
   - Syncs ticket-linked, ephemeral progress snapshots to an external Cross-Repo Hub (e.g., GitHub Wiki).

## 📋 Ticket Creation Conventions
- **No Phase Prefixes in Titles**: Do not include phase identifiers (e.g., `Phase 0:`, `Phase 1.1:`) in ticket/issue titles. Keep titles concise, descriptive, and action-oriented. Use labels/tags to associate tickets with roadmap phases.

## 🚀 Quickstart: Bootstrapping the Agent

Prompt your agent with the following to initialize or audit this workspace:

```
Read the root `AGENTS.md` file and execute the setup workflows in:
1. `.agents/skills/agent-bootstrap/SKILL.md`
2. `.agents/skills/workspace-memory/SKILL.md`

Confirm when workspace setup and audits are complete.
```

# Nano Workspace

Two independent repos under `~/dev/nano-privacy/`, each a personal fork with an
`upstream` remote. Treat them as separate projects despite shared lineage.

## Repos

| Dir            | origin (your fork)                              | upstream                          |
| -------------- | ----------------------------------------------- | --------------------------------- |
| `nano-node/`   | github.com/ATXMJ/nano-node                      | github.com/nanocurrency/nano-node |
| `rsnano-node/` | github.com/atxmj-rsnano/rsnano-node             | github.com/rsnano-node/rsnano-node |

`rsnano-node` lives under the `atxmj-rsnano` org (not `ATXMJ`) because GitHub's
fork network blocks a second fork of the same network in one account —
`rsnano-node` is itself a fork of `nano-node`, so `ATXMJ` already occupies that
network with `ATXMJ/nano-node`.

## Working with the repos

Periodically merge upstream into both forks to stay current and keep clean
attribution:

```bash
git fetch upstream
git checkout develop
git merge upstream/develop   # or: git rebase upstream/develop
git push origin develop
```

### nano-node build notes

- CMake build. Minimal system deps: `build-essential g++ wget python3
  zlib1g-dev cmake git` (authoritative list: `nano-node/docker/ci/Dockerfile-base`;
  RHEL list: `nano-node/util/build_prep/rhel/prep.sh.in`). Add Qt5 deps only for
  `-DNANO_GUI=ON`.
- **Submodules are NOT initialized after clone.** Run before first build:
  ```bash
  git submodule update --init --recursive   # 14 submodules: boost, rocksdb, lmdb, cryptopp, ...
  ```
- Standard build (`ci/build.sh`): `mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=Debug -DPORTABLE=ON -DACTIVE_NETWORK=nano_live_network -DNANO_TEST=OFF -DNANO_GUI=OFF .. && cmake --build . --parallel $(nproc)`

### 🛠️ Sub-Repo Justfile & Execution Directives

Builds, tests, and code quality workflows are managed locally within each sub-repository via dedicated `Justfile` and `AGENTS.md` configurations:

- **C++ Node (`nano-node/`)**: See [nano-node/AGENTS.md](file:///home/mj/dev/nano-privacy/nano-node/AGENTS.md) and `nano-node/Justfile` for C++ build targets, GoogleTest execution (`just test`), and formatting.
- **Rust Node (`rsnano-node/`)**: See [rsnano-node/AGENTS.md](file:///home/mj/dev/nano-privacy/rsnano-node/AGENTS.md) and `rsnano-node/Justfile` for Rust cargo test execution (`just test`), linting (`just lint`), and crate checking.
- **Root-Level Justfile Policy**: Build, test, and lint tasks should remain scoped to their respective sub-repositories (`nano-node/` and `rsnano-node/`). A root-level `Justfile` may be introduced later only if justified for orchestration workflows that fall outside the scope of individual sub-repositories (e.g. cross-repository testnet deployment harnesses or multi-repo release tooling).

See `README.md` for the fuller human-facing writeup.




