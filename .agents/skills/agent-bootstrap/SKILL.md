---
name: agent-bootstrap
description: Automated workspace architecture auto-wiring and dual-documentation (README.md + AGENTS.md) auditing and scaffolding.
---

# 🤖 Agent Bootstrap & Scaffolding Skill

This skill defines the agent-agnostic workflow for:
1. **Workspace Architecture Auto-Wiring**: Auditing and mapping `.agents/skills/` to the agent engine's proprietary settings/directories.
2. **Dual-Documentation Enforcement**: Auditing and scaffolding `README.md` (Human Context) and `AGENTS.md` (Agent Context) across workspace modules.

---

## 🚀 Quickstart Prompt for Humans

To instruct an AI coding agent to bootstrap or audit skills and dual-documentation in any repository containing this skill, copy and send the following prompt:

```text
Read `.agents/skills/agent-bootstrap/SKILL.md` and execute Phase 1 and Phase 2 for this repository.
```

---

## ⚙️ Protocol & Execution Directives

### Phase 1: Skill Audit & Auto-Wiring Protocol

Upon initialization or when running workspace setup, execute the following steps:

#### Step 1: Audit Master Skills Directory
Inspect `.agents/skills/` for all available skill directories containing `SKILL.md`.

#### Step 2: Auto-Wire Engine Proxies / Shims
Map all unlinked skills to your agent engine's proprietary configuration directory (e.g. `.claude/skills`, `.gemini/skills`, `.cursor/rules`, etc.):
- **Primary Strategy (References over Symlinks)**: Avoid OS-level symlinks when possible to prevent cross-platform permission errors. Prefer native file imports (e.g., using `@` references), generating lightweight proxy `.md` or `.mdc` files in proprietary directories, or updating engine configuration files to point to `.agents/skills/`.
- **Fallback Strategy**: Only if native references or proxy files are strictly unsupported by your engine, create directory symlinks (e.g., `ln -s .agents/skills .claude/skills`).
- **Rule**: Never duplicate skill logic or copy `SKILL.md` contents into proprietary folders. Always use references, shims, or symlinks to maintain `.agents/skills/` as the single source of truth.

---

### Phase 2: Dual-Documentation Audit & Scaffolding Protocol

#### Step 1: Audit Directory Context Coverage
Inspect the workspace root and distinct sub-modules/directories to ensure the "Dual-Documentation" rule is satisfied. Every module or directory serving a distinct architectural purpose MUST contain both:
1. **`README.md` (Human Context)**: Focus on the *Why*—high-level architecture, module purpose, visual diagrams, dependency rationales, and human quickstarts.
2. **`AGENTS.md` (Agent Context)**: Focus on the *How*—programmatic build scripts, exact testing commands, localized linting overrides, and strict terminal execution steps.

#### Step 2: Minimal Redundancy & Alignment Rules
- **No Instruction Overlap**:
  - GUI steps, browser interactions, and subjective design decisions belong in `README.md`.
  - Terminal execution commands, build targets, AST parsing, and strict syntax enforcement belong in `AGENTS.md`.
- **Architectural Alignment**:
  - Before performing structural refactoring or module scaffolding, agents MUST read the local `README.md` first to align technical execution with human design intent.
- **Scaffold Missing Context**:
  - If a module or sub-directory lacks `README.md` or `AGENTS.md`, generate initial stubs based on local code context.
