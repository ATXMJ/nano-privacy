---
name: workspace-memory
description: Automated setup, bootstrapping, and daily management of repo-level memory (MEMORY.md), decision logs (DECISIONS.md), and generalized cross-repo progress snapshots.
---

# 🧠 Workspace Memory & State Management Skill

This skill defines an agent-agnostic workflow for bootstrapping and managing:
1. **Repo-Level Operational Memory (`MEMORY.md`)**
2. **Repo-Level Architectural Decision Records (`DECISIONS.md`)**
3. **Cross-Repo Progress & Context (`Progress-Snapshot` & `Cross-Repo-Context` on an external Hub)**

---

## 🚀 Quickstart Prompt for Humans

To instruct any AI coding agent to bootstrap or audit workspace memory in a repository containing this skill, copy and send the following prompt:

```text
Read `.agents/skills/workspace-memory/SKILL.md` and execute Phase 1 for this repository.
```

---

## ⚙️ Protocol & Execution Directives

### Phase 1: Bootstrapping & Setup Workflow

When commanded to bootstrap or audit workspace memory (or upon initializing a new repository), the agent MUST walk the user through an interactive, step-by-step setup process:

#### Step 1: Interactive Ecosystem Interview
Inspect `MEMORY.md` for existing configuration. If not configured, prompt the user to specify:
1. **Issue / Ticket Tracker Platform**:
   - Ask which task management platform is used (e.g. GitHub Issues, Linear, Jira, GitLab Issues, Shortcut).
   - Confirm ticket/issue identifier format (e.g., `#123`, `ENG-456`, `PROJ-789`).
2. **Cross-Repo Documentation Hub**:
   - Ask where cross-repo context & progress snapshots live (e.g. GitHub Wiki, GitLab Wiki, Shared Git Docs Repo, Notion, Confluence).
   - Record the repository URL or workspace/page identifier.

#### Step 2: Tooling & Integration Audit (MCPs, Skills & Plugins)
1. **Audit Agent Capabilities**: Inspect currently connected MCP servers, native tools, and installed skills.
2. **Determine Required Integrations**:
   - **GitHub**: `github` MCP server or `gh` CLI tool.
   - **Linear**: `linear` MCP server.
   - **Jira / Confluence**: `jira` / `confluence` MCP server or API skill.
   - **Notion**: `notion` MCP server.
   - **Git-Based Wiki**: Standard Git CLI execution (`run_command` with git operations).
3. **Guided Setup Walkthrough**:
   - If required MCP servers, plugins, or skills are missing for the user's chosen platform:
     - Inform the user which integrations are missing.
     - Walk the user through setting up the required MCP server config, API tokens, or skills step-by-step.
     - Verify connection once the user completes configuration.

#### Step 3: Persist Ecosystem Selections in `MEMORY.md`
Write the confirmed ecosystem settings under `## ⚙️ Workspace Ecosystem Configuration` in `MEMORY.md` so future agent sessions don't need to re-prompt:

```markdown
## ⚙️ Workspace Ecosystem Configuration
- **Issue Tracker**: GitHub Issues (e.g., `https://github.com/org/repo/issues`)
- **Cross-Repo Hub**: GitHub Wiki (e.g., `git@github.com:org/repo.wiki.git`)
- **Integrations**: `github` MCP server
```

#### Step 4: Scaffold Missing Files & Inject AGENTS.md Directives
- Scaffold `MEMORY.md` and `DECISIONS.md` if missing.
- Ensure the root `AGENTS.md` contains the memory meta-directive block pointing agents to `MEMORY.md`, `DECISIONS.md`, and the Cross-Repo Hub.

---

### Phase 2: Daily Operations & Maintenance Protocol

#### 1. `MEMORY.md` Protocol (Repo-Level Agent Memory)
- **When to read**: Upon session initialization or before executing build/test cycles.
- **What to write**: Workspace ecosystem settings, tool flags, execution quirks, custom environment paths, persistent user preferences, transient session scratchpad notes.
- **What NOT to write**: Work item status updates, bug specifications, or feature backlog items.

#### 2. `DECISIONS.md` Protocol (Local ADRs)
- **Format**: Lightweight Architectural Decision Record (ADR):
  - `ADR-XXX: Title`
  - `Date`, `Status` (Proposed/Accepted/Deprecated), `Related Ticket/Issue` (URL or Identifier like `#123`, `ENG-456`, `PROJ-789`)
  - `Context`, `Decision`, `Consequences`
- **Redundancy Rule**: Link directly to the Issue / Ticket Tracker for background context; do not duplicate problem statements, specs, or acceptance criteria.

#### 3. Cross-Repo Hub `Progress-Snapshot` Protocol
- **Format**: Three dynamic sections:
  1. `Recently Completed`
  2. `Currently in Flight`
  3. `Upcoming Priorities`
- **Ticket / Issue Pointers Only**: Format items concisely with ticket pointers:
  - Example (GitHub): `- [x] **[repo-name]** Short summary - [#123](https://github.com/org/repo/issues/123)`
  - Example (Linear): `- [x] **[repo-name]** Short summary - [ENG-456](https://linear.app/org/issue/ENG-456)`
  - Example (Jira): `- [x] **[repo-name]** Short summary - [PROJ-789](https://jira.org/browse/PROJ-789)`
- **Rolling Window Purge**:
  - Limit `Recently Completed` to a maximum of 5 items.
  - Delete completed items older than 7 days.
  - Purge completed items from `Currently in Flight` once moved to `Recently Completed`.
