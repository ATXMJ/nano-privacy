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

## ADR-002: Relocate All Value-Mixing Off Mainnet L1 (Roadmap v2.0 Architecture Pivot)
- **Date**: 2026-07-30
- **Status**: Accepted
- **Related**: `PRIVACY_ROADMAP.md` v2.0 (supersedes v1.0 L1-first plan)

### Context
Roadmap v1.0 specced deep mainnet L1 changes: stealth-address block fields (Phase 1), confidential Pedersen-committed balances with a plaintext "Declared Weight Dial" for ORV/QoS (Phase 2), and an L1 shielded pool behind a Turnstile (Phase 3). Review against three binding constraints invalidated this approach: (1) existing mainnet nodes must never become "mixers" (legal-risk perception deters the broad casual operator base ORV depends on); (2) impact to core properties must be minimal, with zero fees and supply rigidity non-negotiable; (3) changes to existing third-party wallets are infeasible until the project has momentum, so they must come last. Additional technical findings: L1 stealth addresses invert Nano's O(1) wallet scanning into O(global throughput) trial-ECDH and require both payment ends to upgrade; the Declared Weight Dial's dominant strategy (declare w ≈ balance for priority/weight) erodes the privacy it funds; confidential-balance surgery touches `rep_weights`, `bucketing.cpp`, and `ledger::block_priority()` for Mimblewimble-tier privacy that is empirically weak; nullifier double-spends form a new conflict class that ORV's root-keyed elections cannot arbitrate without novel design work.

### Decision
Mainnet L1 remains byte-for-byte unchanged. Privacy ships in ordered, quarantined layers:
1. **Phase 1 — Network-layer privacy** (Tor/I2P transport, Dandelion++-style stem/fluff): non-consensus, upstreamable, protects all users.
2. **Phase 2 — Chaumian ecash L2** (Cashu/Fedimint-style mints, FROST threshold Ed25519 custody, public reserve accounts + proof-of-liabilities): zero L1 footprint; we ship our own wallet.
3. **Phase 3 — Standalone privacy network** (`nano_privacy_network` reframed from staging environment to *permanent* shielded sister chain) + **federated FROST 1:1 bridge**: nullifier/ORV design doc gates all code; curve trees (FCMP++-style, native Ed25519) primary proving candidate, Halo2 fallback; Penumbra-style batched delegation for shielded ORV weight (v1: shielded funds don't vote); anti-spam = elevated fixed PoW tier (price) + bounded verification lane (cap), no fees; Rust-native prototyping in `rsnano-node`, no FFI until proven.
4. **Phase 4 — Audits and staged launch.**
5. **Phase 5 — Wallet-ecosystem hygiene** (HD rotation, receive jitter, mint/bridge SDK integrations): deliberately last, after momentum.

### Consequences
- **Pros**: Mainnet impact on latency/TPS/fees/node-cost/supply is zero by construction; worst-case ZK failure is bounded to voluntarily-bridged escrow funds; mixing exposure confined to self-selected, Tor-shielded operators/federations; every phase ships standalone durable value with nothing thrown away.
- **Cons**: Bridge federation is an accepted trust bottleneck (mitigated by k-of-n FROST, public auditability, caps, long-term validity-proof exit path); anonymity set is fragmented vs. a single L1 pool; shielding UX is a cross-network hop rather than a native block type.
