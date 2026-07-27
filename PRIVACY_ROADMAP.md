# Nano Privacy Implementation Roadmap

## Executive Summary
This document serves as the foundational technical blueprint for implementing privacy on the Nano block-lattice. It outlines a sequential execution plan designed for a solo AI Platform Engineer/Architect leveraging local agentic AI tools and modular development environments. 

The architecture strictly preserves Nano’s core invariants: zero-fee transactions, deterministic sub-second finality, 133.2M supply rigidity, and the Time-and-Balance (LRU/Bucketing) QoS anti-spam scheduler. The approach focuses first on deep L1 C++ consensus and cryptographic protocol modifications deployed onto an independent experimental network target (`nano_privacy_network`), deferring off-chain ecosystem utilities and L2 solutions to the final phase.

---

## Phase 0: AI Orchestration & Modular Dev Environment Setup

Before writing production code, the development environment must be orchestrated to handle cross-language context (C++ for the Nano node, Rust for ZK-circuits) and massive codebase refactoring.

*   **Modular Architecture:** Isolate the Nano C++ node daemon (`nano-node`) from the cryptographic proving libraries. Create a local FFI (Foreign Function Interface) bridge between the C++ consensus layer and the Rust cryptographic libraries.
*   **Standalone Network Configuration (`nano_privacy_network`):** Configure a distinct build target in CMake and Rust with custom P2P network magic bytes, unique ports (e.g., Peering: 7077, RPC: 7078), and a dedicated genesis block script (allocating the 133.2M supply to a development genesis key). This enables lightweight (~0MB initial DB size), completely isolated testing without requiring mainnet ledger snapshots or risking mainnet collisions.
*   **Agentic AI Tooling:** Deploy a local agentic framework using Claude Opus (or equivalent frontier models) with direct file-system read/write access. 
*   **Continuity Bible:** Establish a strict system prompt and continuity document for the AI agent detailing Nano's specific invariants (Open Representative Voting, Time-and-Balance QoS, account-chain structure). The agent must check all generated code against this rulebook to prevent regressions.

---

## Phase 1: L1 Stealth Addresses

**Objective:** Eliminate receiver address reuse at the protocol level.

### 1.1 State Block Restructuring
*   **Reasoning:** To execute an Elliptic Curve Diffie-Hellman (ECDH) exchange on-chain, the sender needs to pass an ephemeral public key to the receiver.
*   **Implementation:**
    *   Modify `nano/lib/blocks.hpp` to introduce a `v2_state_block`.
    *   Add an optional 32-byte `ephemeral_key` field and a 1-byte `view_tag` (to prevent the receiver from having to perform ECDH math on every transaction on the network).
    *   Ensure backward compatibility for legacy state blocks.

### 1.2 ECDH Key Derivation & Node Integration
*   **Implementation:**
    *   Implement the dual-key stealth address logic using Nano's native Ed25519/Blake2b cryptography. 
    *   The sender generates $R$ (ephemeral key) and derives the one-time destination address.
    *   The receiver scans network traffic, using the `view_tag` to filter, and uses their private view key to detect matches.

### 1.3 Testing & Validation
*   **AI Simulation:** Instruct the agent to generate Python scripts that simulate high-velocity network traffic containing both standard and stealth blocks.
*   **Validation:** Profile the node's block ingestion rate. The added ECDH math on the *sender/receiver wallet side* must not impact the *node's* block validation and signature checking speed. Finality must remain at 300–500ms.

---

## Phase 2: Confidential Balances & ORV Dial

**Objective:** Hide transaction amounts and balances on L1 while maintaining Open Representative Voting (ORV) consensus and Time-and-Balance QoS priorities.

### 2.1 ZK-Circuit Design (Rust)
*   **Reasoning:** Amounts must be hidden without allowing undetectable inflation. 
*   **Implementation:**
    *   Implement Pedersen Commitments for all account balances.
    *   Use Bulletproofs+ for range proofs (proving the committed balance is $\ge 0$). 
    *   **The Declared Weight Dial:** Design a specific ZK-circuit where the user inputs their hidden balance and a plaintext declared weight $w$. The circuit outputs a proof verifying that $w \le \text{balance}$.

### 2.2 The Turnstile Boundary
*   **Reasoning:** A catastrophic bug in the ZK-circuit must not inflate the core $133.2\text{M}$ XNO supply. 
*   **Implementation:** 
    *   Create a specialized state block representing the "Turnstile" contract. 
    *   Moving transparent XNO to a Confidential Balance deducts from the transparent supply and adds to the Turnstile's known aggregate balance. The system explicitly rejects any transaction attempting to withdraw more transparent XNO from the Turnstile than currently exists inside it.

### 2.3 QoS Scheduler Refactoring
*   **Reasoning:** Nano's scheduler needs to know balances to place transactions in QoS buckets. 
*   **Implementation:** 
    *   Modify the Fair Queueing scheduler in `nano/node/scheduler.cpp`. 
    *   Instead of reading the account's actual balance, the scheduler reads the *plaintext declared weight* $w$ verified by the ZKP.
    *   Accounts declaring $w = 0$ (maximum privacy) are routed to the lowest priority dust bucket. 

### 2.4 Testing & Validation
*   **Fuzzing:** Use the AI agent to write aggressive memory fuzzers (using Valgrind and AddressSanitizer) against the FFI boundary between the C++ node and the Rust ZK-prover.
*   **Consensus Testing:** Spin up a local 50-node testnet. Intentionally attempt to submit blocks where $w > \text{balance}$. Verify the network aggressively orphans these blocks and drops the offending peer.

---

## Phase 3: Shielded Pool & QoS Engine

**Objective:** Deploy an opt-in sub-ledger offering absolute sender/receiver/amount anonymity, utilizing advanced scheduler lanes to protect node infrastructure.

### 3.1 Shielded Ledger & Nullifier Architecture
*   **Reasoning:** Account chains inherently leak transaction graphs. A shielded pool uses note commitments and nullifiers to break this graph completely.
*   **Implementation:**
    *   Implement a Halo2 proving system (requiring no trusted setup).
    *   Integrate the nullifier set into the node's ledger database (LMDB/RocksDB).
    *   Wire nullifier double-spend attempts into the existing ORV fork-resolution logic.

### 3.2 ZK-Predicates for the QoS Scheduler
*   **Reasoning:** Shielded notes lack account-level LRU timestamps and visible balances.
*   **Implementation:**
    *   Add a ZK-predicate proving the note's value exceeds a specific bucket floor.
    *   Add a ZK-predicate proving note age by verifying membership against a historical Merkle tree anchor (proving the note existed $T$ blocks ago).

### 3.3 Cost-Weighted Bounded Lanes (Node Defense)
*   **Reasoning:** Verifying a ZK-SNARK takes ~1–3ms. A spammer could exhaust node CPUs without a fee market to deter them.
*   **Implementation:**
    *   Refactor the node's thread pool allocator. 
    *   Assign a compute-cost multiplier to shielded transactions (e.g., $1 \text{ Shielded Tx} = 50 \text{ Transparent Txs}$).
    *   Implement a hard **Bounded Lane**: Restrict the scheduler to spend a maximum of 5% of its CPU cycle time verifying shielded proofs. Excess shielded traffic is queued or dropped, while transparent traffic continues at line speed.

### 3.4 Testing & Validation
*   **Asymmetric Stress Testing:** Use the AI agent to generate massive swarms of valid ZK-proofs and flood the testnet. 
*   **Validation:** Monitor node CPU utilization. The bounded lane must act as a perfect circuit breaker; CPU load should peg at the 5% allocation for shielded verification, and the 300ms finality for standard transparent transactions must remain entirely unaffected.

---

## Phase 4: Audit & Standalone Network Deployment

**Objective:** Final security hardening, peer review, and deployment as an independent standalone network before mainnet upstream proposals.

### 4.1 Automated & Manual Auditing
*   **Agentic Code Review:** Instruct the AI agent to perform a holistic cross-file review looking specifically for C++ memory leaks in the FFI bindings, LMDB database bloat from nullifier sets, and potential concurrency deadlocks in the modified scheduler.
*   **Third-Party Review:** Submit the Halo2 circuits and Turnstile logic to independent cryptographic auditors. 

### 4.2 Independent Network Launch (`nano_privacy_network`)
*   **Implementation:** Deploy seed nodes and initial representative nodes running the `nano_privacy_network` target. Verify live peer discovery, block propagation, and consensus on the separate network infrastructure.

### 4.3 Upstream Integration Proposal (Future Stage)
*   **Implementation:** Once the standalone network is proven stable under real-world conditions, package the modifications as clean PRs for upstream `nanocurrency/nano-node` and `rsnano-node/rsnano-node`, drafting a formal epoch block migration proposal for the main Nano network.

---

## Phase 5: Ecosystem Foundation & L2

**Objective:** Provide additional user privacy through wallet hygiene and off-chain layer-2 protocols without touching the L1 consensus logic. This is deferred until the foundational L1 privacy improvements are established.

### 5.1 Local Wallet Hygiene (Client-Side)
*   **Reasoning:** Nano wallets typically reuse a single address, destroying privacy. Generating unique addresses per transaction mitigates static tracing.
*   **Implementation:** 
    *   Task the AI agent to write automated HD account rotation modules using the standard BIP39/BIP44 derivation paths already native to Nano seeds.
    *   Implement a "receive-block timing jitter" function. Because Nano receives are asynchronous (pending blocks persist indefinitely), the wallet will randomize the delay before publishing the receive block, breaking basic heuristic timing analysis.
*   **Testing:** Unit test the HD derivation against existing Nano RPC endpoints to ensure funds are not stranded in un-scanned index paths.

### 5.2 Federated Chaumian Ecash (L2)
*   **Reasoning:** Provides instant, information-theoretic privacy for users who need it, taking advantage of Nano's feeless L1 to handle instant mint rebalancing.
*   **Implementation:**
    *   Fork a standard Fedimint or Cashu implementation.
    *   Develop a bridge utilizing a multi-signature Nano L1 account. Users deposit XNO to the multi-sig, and the mint issues blind-signed XNO-denominated ecash tokens. 
*   **AI Agent Workflow:** Have the agent translate standard Bitcoin Lightning-based mint deposit logic to interact with Nano's WebSocket subscription and RPC (specifically polling for `history` and `pending` blocks).