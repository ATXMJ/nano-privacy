# 🤖 Agent Bootstrap & Scaffolding Skill

> **Human Documentation & Quickstart Guide**  
> For agent execution instructions, see [`SKILL.md`](file://.agents/skills/agent-bootstrap/SKILL.md).

---

## 🎯 Purpose

This skill standardizes how AI coding agents:
1. **Auto-Wire Agent Skills**: Connect workspace skills from `.agents/skills/` to the agent runtime engine (Claude, Cursor, Antigravity, Windsurf, etc.) using lightweight references/shims.
2. **Enforce Dual-Documentation**: Maintain both `README.md` (Human *Why*) and `AGENTS.md` (Agent *How*) across all architectural modules with zero instruction overlap.

---

## 📐 Architecture & Design Principles

```
module-directory/
├── README.md       <-- Human Context (Why: Architecture, design philosophy, diagrams)
└── AGENTS.md       <-- Agent Context (How: Build steps, test scripts, linting, CLI directives)
```

### Separation of Concerns

| File | Primary Audience | Focus & Content |
| :--- | :--- | :--- |
| **`README.md`** | Human Developers | High-level architecture, module purpose, visual documentation, dependency rationales, and human quickstarts. |
| **`AGENTS.md`** | AI Coding Agents | Programmatic build scripts, exact testing commands, localized linting overrides, and strict execution steps. |

---

## 🚀 Quickstart Prompts for Human Developers

### 1. Bootstrapping Skills & Scaffolding
To instruct your agent to audit skills and verify dual-documentation across your workspace:

```text
Read `.agents/skills/agent-bootstrap/SKILL.md` and execute Phase 1 and Phase 2 for this repository.
```

### 2. Full Workspace Initialization (Bootstrap + Memory)
To perform a complete initialization of both bootstrap protocols and workspace memory:

```text
Read `.agents/skills/agent-bootstrap/SKILL.md` and `.agents/skills/workspace-memory/SKILL.md`, and execute the initialization workflow for this workspace.
```

---

## 🔌 Integration & Tooling Requirements

- **File System Tools**: Read/Write access to local directory tree to inspect and generate `README.md` and `AGENTS.md` files.
- **Engine Shims**: Permission to create references or symlinks in engine-specific configuration folders (e.g. `.claude/skills`, `.gemini/skills`).
