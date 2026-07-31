# Nano Privacy Implementation Roadmap

**Version 2.0** — Revised 2026-07-30. Supersedes v1.0 (L1-first architecture). See `DECISIONS.md` ADR-002 for the rationale behind the pivot.

---

## Executive Summary

This document is the foundational technical blueprint for bringing strong, practical privacy to Nano (XNO) users. It is designed for execution by a solo AI Platform Engineer/Architect leveraging local agentic AI tooling across the two node implementations in this workspace (`nano-node/` C++, `rsnano-node/` Rust).

### The Prime Directive: Mainnet Stays Untouched

Version 1.0 of this roadmap proposed deep L1 consensus modifications (stealth address block fields, confidential Pedersen-committed balances, an L1 shielded pool). Version 2.0 abandons that approach for three binding constraints:

1. **Existing mainnet nodes must never become "mixers."** Requiring every mainnet node to validate, relay, and store anonymized value transfers changes the legal risk profile of node operation — real or perceived — and Nano's security model depends on a broad population of casual node operators and representatives. Value-mixing infrastructure must be quarantined to operators who *knowingly and deliberately opt in*, with their own operator privacy protected.
2. **Impact to Nano's core properties must be as near zero as achievable** — and for two of them, exactly zero:
   - Latency to irreversible finality (~300–500 ms) — minimize impact
   - Throughput (TPS) — minimize impact
   - **Zero fees — non-negotiable**
   - Node operator cost (CPU, disk, bandwidth) — minimize impact
   - **Supply rigidity (133,248,297.920938463463374607170688 XNO fixed, zero inflation) — non-negotiable**
   - Ease of use — minimize impact
3. **Wallet-ecosystem changes come last.** Until the project has visibility and momentum, landing PRs in every third-party wallet is infeasible. Early phases must require **zero changes** from existing wallets; where a client is needed, we ship our own dedicated client.

The architecture that satisfies all three: **the Nano mainnet L1 remains byte-for-byte unchanged.** Privacy is delivered through four quarantined layers, ordered from least to most invasive:

- **Network-layer privacy** in the node software itself (non-consensus, upstreamable — protects everyone).
- **Chaumian ecash (L2)** — information-theoretic payment privacy with zero L1 footprint, via our own dedicated wallet.
- **A standalone privacy network** (`nano_privacy_network`) — a permanent, Nano-derived sister chain hosting the shielded pool, connected to mainnet by a federated 1:1 bridge. This is the "separate network of mixers" whose operators are self-selected and anonymity-protected.
- **Wallet-ecosystem hygiene** — deferred to last, once momentum makes ecosystem PRs viable.

Under this architecture, mainnet impact on latency, throughput, fees, node cost, and supply is **zero by construction** rather than "minimized by careful engineering." The worst-case failure of any zero-knowledge circuit is contained to funds voluntarily placed in the bridge — the mainnet supply invariant is unconditionally outside the blast radius.

### What Was Deliberately Dropped from v1.0 (and Why)

| v1.0 Component | Verdict | Reasoning |
|---|---|---|
| L1 stealth addresses (`v2_state_block`, ephemeral key + view tag) | **Dropped** | Leaves the transaction graph fully public (receive `link` = send hash; exact amounts). Inverts Nano's O(1) wallet scanning economics into O(global throughput) trial-ECDH. Requires *both ends* of every payment to upgrade — an ecosystem-wide wallet coordination we cannot get pre-momentum. Fragments funds across one-time account chains, fighting the time-and-balance QoS scheduler. Stealth addressing comes *free* inside the shielded pool (notes are inherently one-time). |
| L1 confidential balances + Declared Weight Dial | **Dropped** | The dial leaks what it protects: rational users declare `w ≈ balance` to keep QoS priority and ORV weight, so the dominant strategy is to opt out of the privacy. Requires surgery on `rep_weights`, `bucketing.cpp`, and `ledger::block_priority()` — the most consensus-critical accounting in the node. Delivers only Mimblewimble-tier privacy (hidden amounts, visible graph), which is empirically weak (Grin: ~96% of links recovered by flooding attacks). Would put confidential-value handling on every mainnet node, violating the no-mixer constraint. |
| L1 shielded pool with Turnstile | **Relocated** to the standalone privacy network (Phase 3). All the good ideas survive — nullifiers, no-trusted-setup proving, turnstile-style supply accounting, bounded verification lanes, ZK QoS predicates — on a chain where every operator has opted in and a design mistake cannot harm mainnet. |
| C++ ↔ Rust FFI bridge as the primary crypto integration | **Deferred** | Self-inflicted complexity. `rsnano-node` is a full Rust port; the shielded ledger prototype is built natively in Rust first (no FFI to fuzz), then ported to C++ behind a minimal stable C ABI only if/when needed. |
| Upstreaming L1 privacy to `nanocurrency/nano-node` via epoch blocks | **Replaced** | The only upstream proposal remaining on the long-term horizon is a single, narrowly-scoped bridge-verification hook (see Phase 3.6) — proposed only after years of production history, and even that is optional. Network-layer privacy (Phase 1) is upstreamed instead, which is non-consensus and beneficial to all users. |

---

## Phase Overview

| # | Phase | What It Delivers | Mainnet Node Changes | Existing Wallet Changes | Who Bears Mixing Exposure | Depends On |
|---|---|---|---|---|---|---|
| 0 | AI Orchestration & Dev Environment | Agentic tooling, invariant rulebook, standalone network build target, CI | None | None | Nobody | — |
| 1 | Network-Layer Privacy | Tor/I2P transport, relay-origin obfuscation (Dandelion++-style) — defeats first-relay IP deanonymization for **all** Nano users | Yes — **non-consensus**, upstreamable | None | None (IP privacy ≠ value mixing) | 0 |
| 2 | Chaumian Ecash L2 | Information-theoretic payment privacy today, via blind-signature mints backed by Nano multisig; we ship our own wallet | **None** | None | Mint federation only (opt-in, onion-hosted) | 0; benefits from 1 |
| 3 | Standalone Privacy Network + Bridge | `nano_privacy_network` as a permanent shielded sister chain (nullifiers, no-trusted-setup ZK, batched ORV delegation) + federated FROST 1:1 XNO bridge | **None** | None — dedicated client | Privacy-network operators + bridge federation (self-selected, Tor-shielded) | 0, 1; 3a (design doc) gates all 3.x code |
| 4 | Hardening, Audit & Launch | Circuit + bridge audits, adversarial testnets, asymmetric stress testing, mainnet-facing bridge launch | None | None | — | 1–3 |
| 5 | Wallet-Ecosystem Hygiene | HD rotation, receive-timing jitter, one-tap "shield" integrations in **existing** third-party wallets | None | **Yes — deliberately last** | None | Momentum from 1–4 |

**Phase explanations, briefly:**

- **Phase 0** builds the factory before the product: agent tooling, the invariant "Continuity Bible," and the isolated `nano_privacy_network` build target used for all experimentation.
- **Phase 1** attacks the most practical deanonymization vector against Nano *today* — linking transactions to IP addresses via first-relay observation — with pure node-software changes that need no consensus modification and no wallet cooperation. It is the ideal opener: real privacy for everyone, upstreamable, and it builds the maintainer relationships later phases need.
- **Phase 2** delivers the strongest privacy *guarantee* in the entire roadmap (information-theoretic blind-signature unlinkability) at the lowest cost: zero L1 footprint, zero node changes, a self-shipped wallet. Nano's feeless instant L1 is arguably the best mint-rebalancing rail in existence — this is a differentiated, shippable win while the deep R&D of Phase 3 proceeds.
- **Phase 3** is the flagship: a Nano-derived sister chain whose native unit is 1:1 bridged XNO, hosting a full shielded pool (hidden sender, receiver, and amount; whole-pool anonymity set). All consensus experimentation — nullifier conflict resolution under ORV, shielded QoS, bounded verification lanes — happens here, where operators have self-selected and mainnet is unreachable by any failure.
- **Phase 4** is security hardening and the public launch of the bridge + privacy network under audit.
- **Phase 5** closes the loop with the existing wallet ecosystem once phases 1–4 have generated the credibility to land PRs: address hygiene, timing-analysis countermeasures, and integrated on-ramps to the ecash mints and privacy network.

---

## Phase 0: AI Orchestration & Modular Dev Environment

**Objective:** Orchestrate the development environment for cross-language work (C++ `nano-node`, Rust `rsnano-node`, Rust ZK circuits) and establish guardrails that prevent AI-generated regressions against Nano's invariants.

### 0.1 Workspace & Agent Tooling
- Maintain the two-fork workspace per `AGENTS.md` (`ATXMJ/nano-node`, `atxmj-rsnano/rsnano-node`), with periodic upstream merges to stay current.
- Deploy local agentic tooling (Claude Opus-class models) with file-system access, scoped per sub-repo via each repo's `Justfile` + `AGENTS.md` (build: `just build-tests`, test: `just test`, jobs=4 to avoid Debug-link OOM per `MEMORY.md`).
- Track all work as GitHub Issues in the established trackers (`ATXMJ/nano-privacy` for roadmap-level, sub-repo trackers for implementation), labels for phases — no phase prefixes in titles.

### 0.2 The Continuity Bible (Invariant Rulebook)
Establish a strict system-prompt document the agent must check all generated code against. Minimum contents:
- **Consensus:** Open Representative Voting; elections keyed by *root* (account, previous); block cementing semantics; fork resolution and rollback behavior.
- **Anti-spam:** Nano has **no dynamic PoW** — a tiny fixed PoW per block type/epoch (`work_thresholds`, e.g. epoch_2 receive at 1/8 of send difficulty) acts as minimal deterrent; the real defense is the **time-and-balance bucketing QoS scheduler** (`bucketing.cpp` maps balance → bucket index; `ledger::block_priority()` uses max(balance, previous balance) + previous-block sideband timestamp for LRU ordering within buckets).
- **Ledger:** account-chain structure (one operation per block per account); asynchronous receives (pending entries persist indefinitely); receive `link` = source send hash; ledger pruning support (`enable_pruning`).
- **Economics:** zero fees; fixed fully-distributed supply; `rep_weights` exact per-representative accounting updated on every block including rollbacks.
- **The Prime Directive:** no phase before 5 may require changes to mainnet consensus, mainnet block formats, or existing third-party wallets.

### 0.3 Standalone Network Build Target (`nano_privacy_network`)
- Distinct CMake and Cargo build targets with custom P2P magic bytes, unique ports (Peering: 7077, RPC: 7078, WebSocket: 7079), dedicated genesis script.
- **Genesis note (v2 change):** the privacy network's native units represent *bridged* XNO. Genesis allocates the full possible supply to a **bridge-escrow genesis account** controlled by the bridge federation contract logic (Phase 3.5), not to a development key. Circulating supply on the privacy network at any time equals exactly the XNO locked in the mainnet bridge account — the sister chain mints nothing and can never exceed mainnet backing.
- ~0 MB initial DB; completely isolated from mainnet and beta networks; enables 50+ node local testnets in CI.

### 0.4 CI & Mandatory Baseline Regression Suite
To guarantee mainnet consensus, economic, and performance invariants, **every phase of implementation must pass the mandatory baseline regression suite** across both C++ (`nano-node/`) and Rust (`rsnano-node/`) repositories without regressions:

| Sub-Repo | Command | Purpose / Target Area | Baseline Passing Criteria |
| :--- | :--- | :--- | :--- |
| `nano-node/` | `just test` | Core primitives, ledger, cryptography, block stores, IPC | 1,350+ tests passing (0 failures) |
| `nano-node/` | `just test-rpc` | RPC integration and API endpoint verification | 200+ tests passing (0 failures) |
| `nano-node/` | `just test-system` | Multi-node cluster discovery & in-memory system harness | 8+ tests passing (0 failures) |
| `nano-node/` | `just test-consensus` | Elections, voting policies, representative weights & QoS | 240+ tests passing (0 failures) |
| `rsnano-node/` | `just test` | Full Rust workspace unit & integration test suite | All workspace tests passing |
| `rsnano-node/` | `just lint` | Clippy lint & strict static analysis checks | Zero warnings/errors |

Every phase appends its specialized test suite to CI; no feature branch merges with a red baseline.

**Exit criteria:** standalone network target builds in both repos; two-node local network reaches consensus on a test send/receive; Continuity Bible reviewed and pinned; mandatory baseline regression suite 100% green.

---

## Phase 1: Network-Layer Privacy (Upstreamable)

**Objective:** Defeat first-relay IP deanonymization — today the most practical attack against Nano user privacy — with non-consensus node-software changes that protect **all** users, including every future user of Phases 2–3.

**Why first:** It is the only meaningful privacy gain that is pure node software: no consensus change, no wallet change, no new network, no mixing exposure (IP privacy is legally and technically distinct from value mixing — Bitcoin Core ships Tor support; Dandelion++ was a Bitcoin BIP). It is genuinely upstreamable to `nanocurrency/nano-node` and `rsnano-node/rsnano-node`, building the credibility and maintainer relationships that Phase 3.6 and Phase 5 will need.

### 1.1 Threat Model & Baseline Measurement
- Document the adversary: a passive observer running multiple well-connected listening nodes, timing first-seen block announcements to estimate origin IP; plus ISP-level observers.
- Instrument a local testnet to measure baseline origin-inference accuracy (the AI agent generates traffic simulations and triangulation tooling). This becomes the metric Phase 1 must demonstrably degrade.

### 1.2 Anonymous Transport Integration (Tor / I2P)
- Add first-class SOCKS5 outbound support and hidden-service (onion) inbound support to the node's TCP transport layer (`nano/node/transport/`), configurable per-connection-class: `clearnet`, `tor-only`, `hybrid`.
- Publish onion addresses via the existing peering/keepalive machinery (new extensible telemetry field — non-consensus, ignored by old nodes).
- **Latency guardrail:** anonymous transport is opt-in per node. Voting representatives on mainnet default to clearnet to protect the 300–500 ms confirmation invariant; Tor peering is for block *publishing* and for privacy-network nodes (Phase 3), where the finality target is locally relaxed.

### 1.3 Relay-Origin Obfuscation (Dandelion++-Style Stem/Fluff)
- Implement a two-phase relay for locally-originated blocks in `block_rebroadcaster` / publish paths: a **stem phase** (block forwarded along a random path of 1–4 peers, one hop at a time) followed by **fluff** (normal flooding), with per-epoch randomized stem routes and fail-safe timers (if a stem block isn't seen flooded within T seconds, the stem holder fluffs it).
- Critical Nano-specific analysis (agent-assisted): interaction with vote-by-hash and election start timing. Stem delay must stay well under election activation windows so confirmation latency is unaffected; target stem budget ≤ 1 s with fluff fallback.
- Blocks arriving from peers (not locally originated) relay exactly as today — no change to network-wide propagation characteristics or TPS.

### 1.4 Telemetry & Metadata Scrubbing
- Audit node telemetry, RPC defaults, and logs for operator- and user-identifying metadata; add a `privacy_hardened` config profile (minimal telemetry, no address-book leakage, scrubbed logs).

### 1.5 Testing & Upstreaming
- Re-run 1.1's origin-inference harness: demonstrate accuracy degradation to near-random under stem/fluff + Tor.
- Confirm zero regression in confirmation latency and TPS on the local testnet (CI-gated benchmarks).
- Split into minimal, reviewable PRs against upstream `develop` branches: (a) SOCKS5/Tor transport, (b) stem/fluff relay behind a config flag defaulting off, (c) privacy_hardened profile. Engage the Nano Foundation / node maintainers early with the threat-model document.

#### 🧪 Phase 1 Test Verification Protocol
In addition to the **Mandatory Baseline Regression Suite** (`just test`, `just test-rpc`, `just test-system`, `just test-consensus`), Phase 1 completion requires executing:

1. **SOCKS5/Tor Transport Unit Tests**:
   `cd nano-node && just test "socks5.*:tor.*:transport.*"`
2. **Dandelion++ Stem/Fluff Relay Tests**:
   `cd nano-node && just test "dandelion.*:stem_fluff.*:block_rebroadcaster.*"`
3. **Origin-Inference Degradation Benchmark**:
   Run origin triangulation simulation; confirm origin inference accuracy drops to near-random statistical noise.
4. **Finality & TPS Latency Gate**:
   `cd nano-node && just test-system` (confirm confirmation latency remains within ~300-500ms budget).

**Exit criteria:** origin-inference accuracy reduced to statistical noise on the testnet; latency/TPS benchmarks unchanged; mandatory baseline + Phase 1 test suites 100% green; upstream PRs opened.

---

## Phase 2: Chaumian Ecash L2

**Objective:** Deliver information-theoretic payment privacy to Nano users **now**, with zero L1 footprint, zero node changes, and zero existing-wallet changes — via blind-signature ecash mints backed 1:1 by XNO held in multisig, and a dedicated wallet we ship ourselves.

**Why this ranks so high:** Blind signatures give *unconditional* (information-theoretic, not computational) unlinkability between deposit and spend — stronger than any ZK construction in this roadmap. The trade is custodial trust in a mint federation, which is exactly the trade the majority of casual privacy-seeking users already accept (they hold funds on exchanges). Nano's feeless, sub-second L1 is the best mint-rebalancing and deposit/withdrawal rail in existence: mints can rebalance liquidity between each other and process user withdrawals instantly and for free, which no Lightning-backed Cashu mint can match.

### 2.1 Protocol Selection & Fork
- Evaluate and fork **Cashu** (NUT specifications, simpler single-mint + multi-mint model) as the base, with **Fedimint** federation concepts layered for mint-operator threshold custody. Decision recorded as an ADR.
- Token design: XNO-denominated ecash notes in power-of-2 raw denominations; blind Diffie-Hellman key exchange (BDHKE) as in Cashu; keyset rotation policy.

### 2.2 Nano Backend for the Mint
- Replace Lightning deposit/withdrawal logic with a Nano backend:
  - **Deposits:** unique deposit account per invoice (mint-side HD derivation — mint-internal, requires nothing from user wallets); detection via WebSocket `confirmation` subscriptions with RPC `pending`/`history` polling fallback.
  - **Withdrawals:** direct L1 sends from mint accounts; instant, feeless.
  - **Federation custody:** mint reserve held under threshold Ed25519 — **FROST** works natively on Nano's curve (no new cryptography, no L1 changes; the aggregate signature is a standard Ed25519 signature to mainnet). k-of-n geographically distributed signers.
- **Proof-of-liabilities/reserves:** because the reserve account is a public Nano account, anyone can verify reserves in real time; implement periodic proof-of-liabilities (Cashu-style epoch commitments) so users can audit that issued ecash ≤ reserves. This keeps the **supply-rigidity story airtight**: ecash is a claim, XNO on L1 is never inflated, and under-collateralization is publicly detectable.

### 2.3 Mint Operator Privacy & Distribution
- Mints run as Tor onion services by default (leveraging Phase 1 transport work for their backing nodes); operator documentation for anonymous hosting.
- Multi-mint architecture from day one: many small federated mints rather than one large one — limits blast radius of any single mint compromise/abscondment and distributes legal surface across self-selected operators, consistent with the no-mainnet-mixers constraint.

### 2.4 Dedicated Wallet Client
- Ship our own wallet (mobile-first, plus CLI for testing): standard Nano wallet functionality + ecash send/receive/melt/mint + multi-mint balance management + Tor connectivity built in.
- This is **our** client — no third-party wallet PRs required. Existing wallets interoperate trivially because deposits/withdrawals are ordinary Nano sends.

### 2.5 Testing & Validation
- Agent-generated integration suites: deposit/withdraw storms against a local `nano_privacy_network`-target testnet and against beta network; mint-crash recovery; double-spend attempts on ecash notes; federation signer-loss drills (k-of-n degradation).
- Economic simulation: mint liquidity under flash withdrawal load (feeless L1 makes bank-run dynamics *safer* — no fee spiral — verify empirically).

#### 🧪 Phase 2 Test Verification Protocol
In addition to the **Mandatory Baseline Regression Suite** (`just test`, `just test-rpc`, `just test-system`, `just test-consensus`, `cd rsnano-node && just test`), Phase 2 completion requires executing:

1. **FROST Threshold Signature Suite**:
   `cd rsnano-node && just test-pkg frost` (verify Ed25519 k-of-n keygen, signing, and signature aggregation).
2. **Ecash Mint Deposit & Withdrawal Suite**:
   `python3 scripts/test_ecash_mint.py` (verify invoice creation, deposit detection via WebSocket, and melt/withdraw L1 transactions).
3. **Ecash Note Double-Spend & Key Rotation Test**:
   Execute simulated note re-spend attempts; confirm instant rejection and audit log emission.
4. **Proof-of-Liabilities Real-Time Audit**:
   Verify automated check confirming `issued_ecash_notes <= multisig_reserve_balance`.

**Exit criteria:** end-to-end private payment (deposit → blind mint → transfer → melt → withdraw) verified on testnet via our wallet; proof-of-liabilities auditable by a third party; mandatory baseline + Phase 2 test suites 100% green; at least 2 independent federated mints running as onion services.

**Impact scorecard:** L1 latency/TPS/fees/supply: untouched. Node operators: untouched. Ease of use: additive (new optional wallet).

---

## Phase 3: Standalone Privacy Network & Federated Bridge

**Objective:** Stand up `nano_privacy_network` as a **permanent** Nano-derived sister chain hosting a full shielded pool — hidden sender, receiver, and amount with a whole-pool anonymity set — whose native unit is XNO bridged 1:1 from mainnet. This is the "separate network of mixers" permitted by our constraints: every operator is a self-selected volunteer who knowingly runs mixing infrastructure, and their own privacy is protected by design.

**Non-negotiable containment property:** mainnet nodes see only ordinary transparent transactions to/from the bridge account. A catastrophic ZK circuit bug can, at absolute worst, misallocate funds *within the privacy network* or drain *the bridge escrow* — it can never mint mainnet XNO. The 133.2M supply invariant is enforced by unmodified mainnet code and is unconditionally outside the blast radius.

### 3.0 Gate: The Nullifier–ORV Conflict Design Document

**No Phase 3 implementation code is written until this document is complete and reviewed.** This is the one place in the roadmap where we are doing *research*, not engineering:

- Nano elections are keyed by **root** (account, previous). Two forks of one account chain share a root and are voted against each other. Two shielded spends of the same note originate from **different accounts/roots** — they are not forks in Nano's sense; both can have independently valid elections in flight simultaneously, and `active_elections` has no mechanism to arbitrate between them.
- The design doc must specify, minimum: (a) a nullifier-keyed conflict class — e.g., elections keyed on `(nullifier)` in addition to root, or a cross-election dependency index that prevents cementing two blocks revealing the same nullifier; (b) interaction with vote hinting, optimistic elections, and bounded backlog; (c) rollback semantics for the nullifier set and commitment tree (mainnet-style pre-confirmation forks/rollbacks are routine — the nullifier DB must roll back atomically with block rollback); (d) bootstrap/cold-sync verification order for commitment-tree anchors; (e) pruning interaction (nullifier set is append-only forever — see 3.4).
- Deliverable: design doc + adversarial review (agent red-team pass + at least one external reviewer familiar with both Nano consensus and shielded protocols).

### 3.1 Proving System Selection
- **Primary candidate: curve trees / FCMP++-style full-set membership proofs** (the Monero next-generation design): built natively over **Ed25519** via the Helios/Selene curve cycle — Nano's exact curve, so note keys and signatures stay in familiar cryptographic territory; **no trusted setup**; anonymity set = the entire pool history; verification a few ms and batchable; arriving audited and production-tested by Monero on roughly our timeline.
- **Fallback: Halo2 / Orchard-style pool** (Pasta curves): production-proven in Zcash since 2022, no trusted setup, mature tooling — at the cost of a curve boundary between note crypto and Nano's Ed25519 account layer.
- Bake-off deliverable (Rust, in `rsnano-node` workspace): prover time, verifier time, proof size, batch-verification throughput on representative hardware; decision recorded as an ADR. **All prototyping is native Rust — no C++/FFI until the design is proven** (the C++ port, if ever needed, goes behind a minimal stable C ABI written once).

### 3.2 Shielded Ledger Architecture (on the privacy network only)
- Note commitments appended to an incremental Merkle/curve tree; nullifier set in the node's store (LMDB/RocksDB column) with atomic rollback per 3.0(c).
- Shielded block types added to the **privacy network's** block format (it is a separate network — no mainnet format constraints, no epoch-block gymnastics). Transparent Nano-style accounts also exist on the privacy network (bridge in/out, representative operation), giving a Zcash-like transparent/shielded duality *on the sister chain*.
- Turnstile-style internal accounting between the sister chain's transparent and shielded domains (the v1.0 Turnstile idea, relocated): the network rejects any unshield that would exceed the shielded domain's aggregate balance — defense-in-depth *inside* the containment boundary, enabling detection of circuit failures long before the bridge is at risk. Scheduled **pool migrations** (Zcash-style turnstiled pool generations every few years) both bound cryptographic rot and allow nullifier/commitment-tree archival to bound node storage growth.

### 3.3 Consensus & QoS on the Privacy Network
- **ORV with batched shielded delegation (Penumbra-style), not a per-account weight dial:** shielded funds designate a representative at shield-time inside the circuit; per-interval **aggregate net delegation flows** per representative are decrypted/revealed and applied to `rep_weights`. Individual balances never leak; consensus still sees exact public weights; the dominant strategy no longer erodes privacy. **v1 simplification:** shielded funds simply don't vote (transparent-side funds and bridge escrow carry consensus weight — analogous to Zcash's shielded pool not securing PoW); batched delegation ships as v2.
- **QoS for shielded traffic — cap *and* price:**
  - **Price:** an elevated **fixed** PoW tier for shielded block types — slots directly into the existing per-type/epoch `work_thresholds` machinery (epoch_2 already differentiates receive at 1/8× send; add a shielded tier at ~8–64× send difficulty, i.e., seconds on consumer hardware). Consistent with Nano's post-dynamic-PoW philosophy: tiny fixed PoW as deterrent, not a fee market. Fees remain zero.
  - **Cap:** a **bounded verification lane** — refactor the block-processor/verification thread allocation so shielded proof verification is capped at a configurable share (default 5%) of CPU cycle time, with a compute-cost multiplier per shielded tx (~50× transparent). Excess shielded traffic queues or drops; transparent traffic (including all bridge operations) continues at line speed. The lane is a circuit breaker; the PoW price prevents free saturation of the lane (a starved lane is itself a privacy attack — it shrinks the anonymity set).
  - **ZK QoS predicates held in reserve, not default-on:** the v1.0 idea of proving note-value-above-bucket-floor and note-age-via-historical-anchor is novel and workable, but every predicate partitions the anonymity set (value-range × age × time). Default scheduling inside the shielded lane is flat (FIFO + per-source fair queue); the predicates are specced, implemented behind a feature flag, and activated only if flat scheduling proves abusable in stress testing.
- **Relaxed finality target:** the privacy network tolerates onion-routing latency — finality target ~2–5 s there. Mainnet's 300–500 ms is untouched (mainnet is untouched).

### 3.4 Node Operator Protections (the "mixer network" operators)
- **Tor/I2P-only peering by default** (Phase 1 transport), onion-service representatives, `privacy_hardened` telemetry profile mandatory, no operator-identifying data in peering or telemetry.
- **Bounded operator cost:** nullifier-set and commitment-tree growth is the one honest permanent cost of shielded designs. Mitigations, all specced up front: pool-generation migrations with archival (3.2), aggressive pruning of spent transparent-side history (`enable_pruning` inherited from mainnet code), documented disk budgets, and CI checks on DB growth per million shielded txs.
- Operator documentation: legal-posture notes, anonymous-hosting guides, resource requirements.

### 3.5 The Federated Bridge (Mainnet ⇄ Privacy Network)
- **Design honesty:** a trustless two-way peg with zero mainnet changes is essentially impossible; v1 is a **federation** — the trust bottleneck of this architecture, accepted knowingly. Mitigations:
  - **Threshold custody:** mainnet escrow account controlled by k-of-n **FROST** Ed25519 (shared machinery with Phase 2.2 — one audited implementation serves both). Signers geographically distributed, onion-operated, ideally overlapping with respected community entities as the project earns support.
  - **Full public auditability:** the escrow is an ordinary public Nano account; privacy-network supply is mintable only against observed escrow deposits (SPV-style proofs of mainnet confirmation validated by all privacy-network nodes — mainnet block headers/votes are cheap to verify since privacy-network nodes can run mainnet light-observers). Invariant checked by every privacy-network node: `circulating sister-chain supply ≤ escrow balance`. Theft is instantly visible; inflation of mainnet XNO is impossible regardless.
  - **Rate limits & circuit breakers:** per-interval unshield caps, time-locked large withdrawals, federation emergency-halt (halts the *bridge*, never mainnet).
  - **User flow:** shield = send XNO to escrow (an ordinary send any existing wallet can make) with a destination tag → federation attests → sister-chain XNO minted to the user's privacy-network transparent account → user shields into the pool. Unshield = reverse. Our Phase 2 wallet gains privacy-network support (still no third-party wallet changes).
- **Long-term exit from federation trust (future work, explicitly out of v2 scope):** a validity-proof light-client bridge would need one minimal, narrowly-scoped mainnet hook (a single "verify aggregate proof against bridge account" primitive via epoch upgrade). This is proposed upstream **only after years of production history**, as a small, optional, self-contained feature — a far easier ask than "merge a shielded pool," and Phase 1's upstream relationship-building is the groundwork.

### 3.6 Testing & Validation
- 50+ node local privacy-network testnets (CI): nullifier double-spend races under adversarial vote scheduling (the 3.0 design's acid test); fork/rollback storms with atomic nullifier-set rollback verification; bootstrap from genesis with commitment-tree verification.
- **Asymmetric stress testing:** agent-generated swarms of *valid* proofs flooding the shielded lane — verify the bounded lane pegs at its allocation, transparent/bridge traffic is unaffected, and the elevated PoW makes sustained saturation expensive; memory/thread fuzzing (ASan/TSan) on the verification pipeline.
- Bridge adversarial drills: signer compromise below and at threshold, attempted over-mint, escrow/withdrawal race conditions, federation-halt and recovery.

#### 🧪 Phase 3 Test Verification Protocol
In addition to the **Mandatory Baseline Regression Suite** (`just test`, `just test-rpc`, `just test-system`, `just test-consensus`, `cd rsnano-node && just test`), Phase 3 completion requires executing:

1. **Nullifier & Atomic Rollback Suite**:
   `cd nano-node && just test "nullifier.*:shielded_ledger.*"` (verify nullifier insertion, double-spend rejection, and atomic ledger rollback).
2. **Proving System Performance Bake-off**:
   `cd rsnano-node && just test-pkg zksnark` (benchmark proof generation and verification latency; confirm verifier execution $\le 3\text{ms}$).
3. **Bounded-Lane CPU Circuit-Breaker Stress Test**:
   `cd nano-node && just test-consensus "*bounded_lane*"` (verify CPU allocation for shielded verification pegs at $\le 5\%$, protecting transparent traffic).
4. **1:1 Bridge Escrow Supply Invariant Verification**:
   `just test-system "bridge_supply_invariant.*"` (verify `circulating_privacy_supply <= mainnet_escrow_balance` under over-mint attack attempts).

**Exit criteria:** design doc (3.0) reviewed; proving-system ADR decided by benchmark; devnet running the full loop (escrow → mint → shield → private transfer → unshield → escrow release) with all invariant checks green; mandatory baseline + Phase 3 test suites 100% green.

---

## Phase 4: Hardening, Audit & Launch

**Objective:** Independent verification of every trust-bearing component, then staged public launch.

### 4.1 Automated & Agentic Review
- Holistic cross-file agent review passes: memory safety in verification pipeline, DB-growth pathologies (nullifier/commitment stores), concurrency deadlocks in the bounded-lane scheduler, rollback atomicity, bridge state-machine completeness.
- Continuous differential testing between `rsnano-node`-based privacy nodes and any C++ port paths.

### 4.2 Third-Party Audits (budgeted, sequenced)
1. FROST threshold-custody implementation (shared by ecash mints and bridge — audit once, deploy twice).
2. ZK circuits and shielded ledger (proving system per 3.1 ADR; if curve trees, coordinate with/lean on Monero-ecosystem audit work).
3. Bridge federation logic and supply-invariant enforcement.
4. Phase 1 network-privacy changes (lighter review; upstream review process itself is a second set of eyes).

### 4.3 Staged Public Launch
- **Stage A:** public privacy-network testnet (bridged to Nano beta network), bug bounty on supply-invariant and nullifier-conflict violations.
- **Stage B:** mainnet-facing bridge launch with conservative caps (low escrow ceiling, small unshield limits), n small federated signers, published audits, real-time public dashboards: escrow balance vs. sister-chain supply, proof-of-liabilities for mints.
- **Stage C:** progressive cap raises tied to incident-free operation milestones.

#### 🧪 Phase 4 Test Verification Protocol
In addition to the **Mandatory Baseline Regression Suite** (`just test`, `just test-rpc`, `just test-system`, `just test-consensus`, `cd rsnano-node && just test`), Phase 4 completion requires executing:

1. **ASan / Valgrind Memory Safety Fuzzing**:
   Run AddressSanitizer memory sanitization across C++ FFI boundaries and proof verification pipelines.
2. **Adversarial Testnet Stress Run**:
   `just test-system "adversarial_network.*"` (50+ nodes under simulated network partitions, high load, and dropped votes).
3. **Public Escrow & Proof-of-Liabilities Dashboard Verification**:
   Automated real-time monitoring assertions confirming live zero-drift between mainnet escrow and sister-chain circulating supply.

**Exit criteria:** all audits closed or risk-accepted with public write-ups; Stage B live; monitoring/alerting proven by drill; mandatory baseline + Phase 4 test suites 100% green.

---

## Phase 5: Wallet-Ecosystem Hygiene (Deliberately Last)

**Objective:** With phases 1–4 shipped and visible, spend the earned credibility on the third-party wallet ecosystem — the changes that were infeasible to coordinate at project start.

### 5.1 Address & Timing Hygiene in Existing Wallets
- **HD account rotation:** automated per-payment account rotation using BIP39/BIP44 paths already native to Nano seeds; reference implementation + PRs to major wallets; gap-limit scanning guidance so funds are never stranded in unscanned indices (unit-tested against standard RPC endpoints).
- **Receive-timing jitter:** because Nano receives are asynchronous (pending persists indefinitely), wallets randomize receive-block publication delay to defeat send→receive timing correlation. Reference module + PRs.
- **Amount-hygiene guidance:** optional round-amount splitting to weaken exact-amount graph matching, documented honestly as a heuristic mitigation, not a guarantee.

### 5.2 Ecosystem Integrations
- One-tap "private send" in existing wallets: melt to a Phase 2 ecash mint, or shield via the Phase 3 bridge — SDKs (Rust core, bindings for TS/Kotlin/Swift) so wallet integrations are thin.
- Wallet-facing documentation of the full privacy menu and each option's honest threat model: network privacy (free, automatic) → ecash (custodial, information-theoretic) → shielded pool (self-custody, computational ZK).

### 5.3 Standards & Advocacy
- Publish the accumulated specs (stem/fluff for Nano, FROST multisig conventions, bridge attestation format, shielded QoS design) as open community RFCs for the broader Nano ecosystem.

#### 🧪 Phase 5 Test Verification Protocol
In addition to the **Mandatory Baseline Regression Suite** (`just test`, `just test-rpc`, `just test-system`, `just test-consensus`, `cd rsnano-node && just test`), Phase 5 completion requires executing:

1. **HD Account Rotation & Gap-Limit Suite**:
   `python3 scripts/test_hd_rotation.py` (verify derivation path scanning and zero stranded funds across RPC calls).
2. **Receive-Timing Jitter Unit Tests**:
   `cd nano-node && just test "timing_jitter.*"` (verify Poisson/randomized publication delay distribution).
3. **SDK Cross-Platform Binding Verification**:
   Verify TS/Kotlin/Swift SDK test bindings against local testnet endpoints.

**Exit criteria:** hygiene modules merged in ≥2 widely-used wallets; SDK adopted by ≥1 third-party wallet for mint/bridge access; mandatory baseline + Phase 5 test suites 100% green.

---

## Cross-Cutting Risk Register

| Risk | Phase | Severity | Mitigation |
|---|---|---|---|
| Bridge federation compromise/abscondment | 3–4 | **High** — the architecture's accepted trust bottleneck | k-of-n FROST, distributed onion-operated signers, public escrow auditability, caps/time-locks/halts, long-term validity-proof exit path; mainnet supply unconditionally safe regardless |
| Nullifier–ORV conflict design flaw (shielded double-spend) | 3 | **High** | Hard gate 3.0: no code before reviewed design doc; adversarial vote-scheduling CI; contained to privacy network by construction |
| ZK circuit soundness bug | 3–4 | High → contained | No trusted setup; internal turnstile detection; pool migrations; audits; worst case bounded to bridge escrow, never mainnet supply |
| Mint abscondment (ecash) | 2 | Medium | Many small federated mints; public reserve accounts; proof-of-liabilities; user guidance to distribute across mints |
| Anonymity-set fragmentation (separate network + multiple mints vs. one L1 pool) | 2–3 | Medium — inherent cost of the no-mainnet-mixer constraint | Ecash serves casual-privacy majority so pool serves the self-custody minority well; single shielded pool (not per-asset pools) on the privacy network; flat in-pool scheduling to avoid predicate partitioning |
| Shielded-lane starvation as a privacy attack (free spam shrinks anonymity set) | 3 | Medium | Elevated fixed PoW price + bounded lane cap; predicates in reserve |
| Stem/fluff interaction with election timing | 1 | Medium | Stem budget ≪ election activation window; fluff fail-safe; CI latency gates |
| Nullifier/commitment storage growth (permanent node cost) | 3 | Medium | Pool-generation migrations + archival; pruning; published disk budgets; DB-growth CI |
| Upstream PRs rejected (Phase 1) | 1 | Low | Forks carry features regardless; privacy network needs them independently; upstream merge is upside, not dependency |
| Wallet ecosystem ignores Phase 5 | 5 | Low | Phases 1–4 build momentum first (the constraint that ordered this roadmap); our own wallet exists as fallback |
| Solo-developer bus factor / scope | all | Medium | Agentic tooling per Phase 0; each phase ships standalone durable value; ADRs + Continuity Bible keep context recoverable |

---

## Invariant Impact Scorecard (Roadmap-Wide)

| Core Property | Impact | Mechanism |
|---|---|---|
| Latency to irreversible finality (mainnet) | **Zero** | Mainnet untouched; stem/fluff CI-gated to no measurable delta; relaxed finality lives only on the sister chain |
| Throughput / TPS (mainnet) | **Zero** | Mainnet untouched; bridge traffic is ordinary transparent txs |
| Zero fees | **Preserved everywhere** (non-negotiable) | No fee market anywhere, including the privacy network — shielded anti-spam is fixed PoW price + bounded lane cap |
| Node operator cost (mainnet) | **Zero** | Mainnet untouched; Phase 1 features opt-in; storage growth confined to self-selected privacy-network operators, and bounded there |
| Supply rigidity / zero inflation | **Preserved unconditionally** (non-negotiable) | Mainnet code unchanged; sister-chain supply ≤ public escrow, checked by every privacy-network node; ecash audited via proof-of-liabilities; worst-case ZK failure bounded to voluntarily-bridged funds |
| Ease of use | **Additive** | Transparent Nano completely unchanged (O(1) wallet scanning preserved — no stealth-address scanning burden); privacy features are opt-in layers with their own clients until Phase 5 folds them into ecosystem wallets |
| Mainnet nodes as mixers | **Never** | All value-mixing confined to opt-in privacy-network operators and mint/bridge federations, each Tor-shielded by design |
