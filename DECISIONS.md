# 🏛️ Architectural Decision Log (ADR)

> [!NOTE]
> Record significant architectural decisions, trade-offs, and design choices here. Keep entries concise and link to relevant issue/ticket tracking items for detailed context.

## ADR-001: Establish Agent Workspace Memory & Cross-Repo Protocol
- **Date**: 2026-07-27
- **Status**: Accepted
- **Related Issue**: N/A

### Context
Coding agents working across sub-repositories (`nano-node` and `rsnano-node`) need a structured, agent-agnostic way to persist operational learnings, architectural decisions, and cross-repository flight status without duplicating issue/ticket descriptions or cluttering Git history.

### Decision
Adopt the `workspace-memory` skill pattern:
1. **`MEMORY.md`**: Store local tool preferences, execution flags, and environment quirks.
2. **`DECISIONS.md`**: Store lightweight Architectural Decision Records (ADRs) linking to ticket/issue trackers (e.g. GitHub Issues, Linear, Jira).
3. **Cross-Repo Hub**: Maintain pointer-only, ticket-linked progress snapshots and cross-repo context on an external documentation platform (e.g. GitHub Wiki), with rolling window purges for completed work.

### Consequences
- **Pros**: Zero redundancy between task specifications (Issue/Ticket Tracker), operational memory (`MEMORY.md`), ADRs (`DECISIONS.md`), and flight status (Hub Snapshot). Standardized via `.agents/skills/workspace-memory/SKILL.md`.
- **Cons**: Requires periodic purging of old completed items from the Cross-Repo Snapshot Hub during updates.
