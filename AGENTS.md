# ⚙️ Agent Initialization Protocol & Meta-Directives

## 🤖 Agent Workspace Architecture & Auto-Wiring

This project implements a universal agent architecture relying on two open standards: [AGENTS.md](https://github.com/agentsmd/agents.md) for hierarchical context and [Agent Skills](https://github.com/agentskills/agentskills) for executable workflows. 

**Your Single Source of Truth:**
*   **Context:** Nested `AGENTS.md` files dictate localized rules, build steps, and syntax constraints for their specific modules or scopes.
*   **Skills:** The `.agents/skills/` directory is the master catalog for all standardized `SKILL.md` workflows.

**Initialization & Skill Audit Instruction:**
Before executing complex workflows (and upon every agent initialization), you must inspect `.agents/skills/` for any new, modified, or unshimmed skills and automatically map them to your native runtime configuration.
*   **Skill Audit & Sync:** Audit `.agents/skills/` to identify skills that lack active shims, references, or links in your engine's proprietary directory. Immediately generate or update shims/proxies for any unmapped skills.
*   **Primary Strategy (References over Symlinks):** Avoid OS-level symlinks when possible to prevent cross-platform permission errors. Prefer native file imports (e.g., using `@` references), generating lightweight proxy `.md` or `.mdc` files in your proprietary directories, or updating your native settings file to point to the master catalog.
*   **Fallback Strategy:** Only if native references or proxy files are strictly unsupported by your engine, fall back to creating directory symlinks (e.g., `ln -s .agents/skills .claude/skills`).

**Rule:** Never duplicate skill logic or copy `SKILL.md` contents into your proprietary folders. Always use references, shims, or symlinks to maintain `.agents/skills/` as the single source of truth.

---

## 🏗️ Agent Scaffolding & Bootstrapping Protocol

This repository enforces a strict "Dual-Documentation" rule. At the module level, or within any directory serving a distinct architectural purpose, you must ensure both context files exist. There should be no directory in this project where context is unclear.

When scaffolding a new module or distinct directory, you must generate both:
1.  **`README.md` (Human Context):** Focus on the *Why*. This covers high-level architecture, module purpose, visual documentation, dependency rationales, and human-readable quickstarts.
2.  **`AGENTS.md` (Agent Context):** Focus on the *How*. This covers programmatic build scripts, exact testing commands, localized linting overrides, and strict terminal execution steps.

**The Minimal Redundancy Constraint:**
You must actively prevent instruction overlap between these two files. 
*   If an instruction requires a human to open a GUI, use a browser, or make a subjective design decision, it belongs in the `README.md`. 
*   If an instruction requires terminal execution, AST parsing, or strict syntax enforcement, it belongs in the `AGENTS.md`.

**Architectural Alignment Rule:**
Because `AGENTS.md` intentionally strips out the philosophical *Why* to optimize your context window, you operate with a designed "context blindspot." For any task involving architectural changes, structural refactoring, or initial module scaffolding, you MUST explicitly read the local `README.md` first to align your technical execution with the human design intent.

## 📋 GitHub Issue Creation Conventions
- **No Phase Prefixes in Titles**: Do not include phase identifiers (e.g., `Phase 0:`, `Phase 1.1:`) in GitHub issue titles. Keep titles concise, descriptive, and action-oriented (e.g., "Establish C++ / Rust FFI Cryptographic Bridge"). Use issue labels (e.g., `phase-0`, `phase-1`) to associate tickets with roadmap phases.


## 🚀 Quickstart: Bootstrapping the Agent

Prompt your agent with the following to instruct it to bootstrap this project.

```
Read the "Agent Initialization Protocol & Meta-Directives" section in the root `AGENTS.md` file. 

Execute the following two steps:
1. Audit `.agents/skills/` for any new, updated, or non-shimmed skills and perform Workspace Architecture Auto-Wiring to map them to your native configuration (preferring references/shims over symlinks).
2. Audit the current working directory. If it is missing the required dual-documentation files, execute the Agent Scaffolding & Bootstrapping Protocol to generate them based on the current context.

Confirm when both steps are complete.
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




