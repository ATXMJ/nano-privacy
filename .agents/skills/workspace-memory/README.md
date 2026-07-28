# 🧠 Workspace Memory & State Management Skill

> **Human Documentation & Quickstart Guide**  
> For agent execution instructions, see [`SKILL.md`](file://.agents/skills/workspace-memory/SKILL.md).

---

## 🎯 Purpose

This skill establishes an agent-agnostic, zero-redundancy pattern for managing context, memory, decisions, and progress across multi-repository workspaces and any AI coding assistant (Antigravity, Claude, Cursor, Windsurf, AutoGPT, etc.).

---

## 📐 Architecture & Design Principles

```
+-------------------------------------------------------------------------------+
|                             GITHUB ISSUES / LINEAR / JIRA                     |
|                   (Single Source of Truth for Task Specs & Backlog)           |
+---------------------------------------+---------------------------------------+
                                        |
                 +----------------------+----------------------+
                 |                                             |
                 v                                             v
+----------------------------------+        +-----------------------------------+
|      REPO ROOT (Local Git)       |        |   CROSS-REPO HUB (External Docs)  |
|                                  |        |   (GitHub Wiki, Notion, Confluence) |
| - MEMORY.md                      |        |                                   |
|   (Tool preferences & quirks)    |        | - Progress-Snapshot               |
|                                  |        |   (Issue-linked flight status;    |
| - DECISIONS.md                   |        |    purged regularly)              |
|   (Architectural Decision Logs   |        |                                   |
|    linking to issue #s)          |        | - Cross-Repo-Context              |
|                                  |        |   (Multi-repo topology & IPC)     |
+----------------------------------+        +-----------------------------------+
```

### Separation of Concerns

| File / Component | Purpose | What Belongs Here |
| :--- | :--- | :--- |
| **Issue Tracker** *(GitHub, Linear, Jira)* | **Work Items** | Comprehensive task specifications, bug reports, acceptance criteria, and discussion threads. |
| **`MEMORY.md`** *(Repo Root)* | **Agent Memory** | Tool flags, build/test command quirks, environment variables, user preferences, transient scratchpad notes. |
| **`DECISIONS.md`** *(Repo Root)* | **Architectural Records** | Lightweight ADRs detailing technical trade-offs, decisions, and consequences. Links to issue/ticket #s for context. |
| **Cross-Repo Hub** *(Wiki/Docs)* | **Flight Status & Topology** | Pointer-only progress snapshots linking to active tickets. Purged regularly to avoid stale content. |

---

## 🚀 Quickstart Prompts for Human Developers

### 1. Bootstrapping a New Repository
To set up workspace memory in a new repo (or audit an existing setup), copy and send this prompt to your AI coding agent:

```text
Read `.agents/skills/workspace-memory/SKILL.md` and execute Phase 1 for this repository.
```

### 2. Recording an Architectural Decision
To record a design trade-off or ADR:

```text
Record an architectural decision in DECISIONS.md for ticket #123 regarding [brief description].
```

### 3. Updating Progress Flight Status
To update the cross-repo flight status:

```text
Update the Cross-Repo Progress Snapshot for our work on ticket #123.
```

---

## 🔌 Integration & Tooling Requirements

To enable your agent to read and update issue trackers and external documentation hubs, ensure the corresponding **Model Context Protocol (MCP) server** or skill is configured in your agent environment:

| Platform | Recommended Integration |
| :--- | :--- |
| **GitHub Issues & Wiki** | `github` MCP server or `gh` CLI |
| **Linear** | `linear` MCP server |
| **Jira & Confluence** | `jira` / `confluence` MCP server or REST API skills |
| **Notion** | `notion` MCP server |
| **Git-Based Wiki** | Standard Git toolsuite (`run_command` with git operations) |
