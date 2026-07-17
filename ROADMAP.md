# Learnault Soroban Contracts Roadmap

This roadmap covers the on-chain protocol required by the TRD: authoritative course achievements, reward distribution, credentials, quests/scholarships, staking, governance, and future private proofs. Course content, lessons, quizzes, and learner PII remain off-chain in the API; contracts store only the identifiers, commitments, policies, and events needed for trust and verification.

## Tracking rules

- `[x]` — the specifically named implementation is present and verified by repository tests or recorded CI evidence; deployment and production hardening are tracked separately.
- `[~]` — implementation exists but is incomplete, unverified after recent merges, too coarse for the final protocol, or not deployed/integrated.
- `[ ]` — pending.
- A phase closes only when every checklist item in that phase is `[x]`.
- A unit-tested function is not production-complete until its cross-contract, deployment, upgrade, and operational assumptions are verified.

Status baseline: 17 July 2026 after merge `17aaf44`. Six contract crates are present under `contracts/`. A current local verification run could not complete because the workstation ran out of disk space during compilation; older CI evidence cannot close post-merge verification items.

## Current implementation inventory

- [~] Course Registry supports course creation, metadata hash, enrollment, a numeric progress counter, final-completion badge/reward calls, status, ownership transfer, events, and upgrade.
- [~] Reward Pool supports one token, approved spenders, funding, distribution, pause, emergency sweep, events, and upgrade.
- [~] Badge NFT supports soulbound badge mint/revoke/query by learner and course ID, events, and upgrade.
- [~] Quest Engine supports build/explore quests, proof hash submission, review/batch review, payout/refund, stake multiplier lookup, pause, events, and upgrade.
- [~] Stake Vault supports stake, a resettable lock, full unstake, fixed multiplier tiers, events, and upgrade.
- [~] Governance supports badge-weighted voting, cancellation, execution marking, events, and upgrade, but exposes no proposal-creation entry point.

## Phase 0 — Workspace, protocol specification, and reproducible builds

**Goal:** establish one buildable workspace and an agreed protocol boundary before contract behavior expands.

### Repository and build foundation

- [x] Create the six Course Registry, Reward Pool, Badge NFT, Quest Engine, Stake Vault, and Governance crates.
- [ ] Fix the root `Cargo.toml` paths or make `contracts/Cargo.toml` the single documented workspace root.
- [ ] Remove/ignore `.DS_Store` and other workstation artifacts and verify a clean Git status after build/test.
- [ ] Pin supported Rust, Stellar CLI, Soroban SDK, target, and formatter/linter versions.
- [ ] Run `cargo fmt`, Clippy with warnings denied, every crate test, cross-contract tests, and every Wasm build after the latest merge.
- [ ] Add all six contracts to CI reporting; publish test counts, Wasm hashes, sizes, and interface/spec artifacts per commit.
- [ ] Add deterministic/reproducible Wasm build verification and release artifact signing.

### Protocol specification

- [ ] Document which state is authoritative on-chain versus in PostgreSQL and how discrepancies are repaired.
- [ ] Define canonical IDs and encodings for course, course version, module/lesson completion commitment, learner wallet, credential, program, quest, and reward policy.
- [ ] Define contract roles: protocol admin, course issuer/instructor, completion verifier, reward spender, credential issuer/revoker, quest reviewer, upgrader, and emergency guardian.
- [ ] Define network/asset configuration, decimal handling, amount bounds, metadata URI/hash format, and domain-separated hashes.
- [ ] Define every contract error code, event schema, storage key, TTL class, and versioning rule.
- [ ] Define deployment dependencies and initialization order across Course Registry, Badge/Credential, Reward Pool, Quest Engine, Stake Vault, and Governance.
- [ ] Produce architecture, threat model, trust assumptions, and sequence diagrams for course completion and quest payout.

**Closure evidence:** one root command reproduces every artifact and the reviewed protocol specification maps each API workflow to explicit contract calls/events.

## Phase 1 — Shared security, configuration, and lifecycle primitives

**Goal:** make authorization, configuration, pause, storage, and upgrades consistent before funds or credentials are expanded.

### Roles and configuration

- [ ] Introduce shared role/config patterns or libraries without creating duplicate exported symbols in dependent contracts.
- [ ] Separate day-to-day verifier/spender/issuer roles from protocol admin and upgrade authority.
- [ ] Add two-step role transfer/acceptance and explicit role removal where loss of access could strand funds or credentials.
- [ ] Add zero-address/equal-address validation and initialization/configuration event coverage.
- [ ] Add contract/config version query functions needed by the API deployment registry.

### Safety controls

- [~] Standardize pause/unpause across Course Registry, Reward Pool, Badge/Credential, Quest Engine, Stake Vault, and Governance with clear read/write behavior.
- [ ] Replace panic strings with stable contract error enums/codes and document client mappings.
- [ ] Add checked arithmetic and explicit lower/upper bounds for IDs, counts, amounts, durations, quorum, fees, multipliers, and batch sizes.
- [ ] Define storage TTL values and bump instance/persistent entries during supported operations.
- [ ] Add explicit migration hooks/version checks for storage-layout upgrades.

### Upgrade and emergency operations

- [~] Standardize upgrade authorization and `ContractUpgraded` events across all six contracts.
- [ ] Put upgrade authority behind the approved multisig/timelock policy and define emergency guardian limitations.
- [ ] Add pre-upgrade compatibility checks, post-upgrade version checks, and tested rollback/migration procedures.
- [ ] Define emergency recovery behavior separately for user-owned stake, employer escrow, reward treasury, and non-transferable credentials.
- [ ] Test compromised verifier, compromised spender, compromised admin, paused dependency, failed upgrade, and expired TTL scenarios.

**Closure evidence:** all contracts use reviewed role/error/TTL/upgrade conventions and lifecycle tests pass across contract boundaries.

## Phase 2 — Course registry, verified learning completion, and credential protocol

**Goal:** turn the current numeric progress counter and basic badge into a precise, versioned, verifiable learning record.

### Course registry model

- [x] Implement admin initialization and authorized course creation with non-zero module count.
- [x] Implement course lookup/count, activation/deactivation, metadata-hash update, and related events.
- [x] Implement learner enrollment with missing-course and duplicate-enrollment rejection.
- [x] Implement instructor ownership transfer and authorize the new instructor to update metadata.
- [x] Implement learner progress and finished-state queries for the current numeric-counter model.
- [ ] Add course version, canonical metadata commitment/URI, issuer, completion policy hash, reward policy ID, credential schema ID, and created/updated timestamps.
- [ ] Add explicit course version publication/freeze so earned credentials always reference immutable requirements.
- [ ] Add course retirement/deprecation without invalidating previously issued credentials.
- [ ] Add query methods/events needed to resolve current and historical course versions.

### Completion verification

- [x] Implement authorized numeric progress increments with final-module bounds and module/course completion events.
- [ ] Replace blind `complete_module` counter increments with unique module/requirement IDs or a signed completion commitment tied to course version.
- [ ] Prevent replay of the same module/requirement while allowing distinct required items in any policy-approved order.
- [ ] Bind completion authorization to verifier, learner, course, course version, requirement ID, result commitment, nonce, and expiry.
- [ ] Define whether enrollment is required on-chain and enforce it consistently before progress mutation.
- [ ] Store or emit enough information for the API/indexer to prove which requirements were completed without putting quiz answers or PII on-chain.
- [ ] Calculate final completion only from the immutable completion policy and make completion idempotent.
- [ ] Add batch completion only if gas/size benchmarks show a clear benefit and batch failure semantics are defined.

### Credential/badge model

- [x] Implement registry-authorized soulbound badge minting with duplicate prevention and mint events.
- [x] Implement authorized badge revocation and revoke events.
- [x] Implement learner badge list/count and learner-course badge existence queries.
- [ ] Replace the minimal `{course_id, timestamp}` badge with a versioned credential record containing credential ID, course/version, issuer, holder, metadata commitment, issued ledger/time, and status.
- [ ] Add direct credential lookup and pagination/index-friendly queries rather than returning an unbounded learner vector.
- [ ] Add revocation reason commitment, revocation timestamp, issuer authorization, and status verification.
- [ ] Define reissue/supersede behavior for corrected metadata or wallet migration without erasing history.
- [ ] Add issuer rotation and holder-wallet migration/recovery rules with explicit consent and audit events.
- [ ] Define premium certificate versus achievement badge schemas without making paid status alter learning truth.

### Cross-contract completion

- [x] Trigger configured Badge NFT minting only on final course completion.
- [x] Trigger configured Reward Pool payout only on final course completion.
- [x] Persist progress before cross-contract calls and test configured, unconfigured, unauthorized, duplicate, and multiple-learner cases.
- [ ] Remove the hard-coded `10 USDC` reward and resolve a versioned course/program reward policy.
- [ ] Make badge and reward outcomes separately observable and retryable when a configured dependency fails.
- [ ] Define atomic versus eventual behavior: completion must never be lost or duplicated because badge/reward contracts are paused or underfunded.
- [ ] Add integration tests for repeat module IDs, stale versions, invalid signatures, duplicate finalization, dependency pause/failure, badge retry, reward retry, and event replay.

**Closure evidence:** a unique set of verified requirements completes one immutable course version and produces one status-verifiable credential without leaking lesson or quiz data.

## Phase 3 — Reward treasury, program budgets, referrals, and scholarships

**Goal:** evolve the single-token payout pool into auditable, policy-controlled incentive infrastructure.

### Reward Pool hardening

- [x] Implement authenticated single-token pool initialization and pool-funded events.
- [x] Implement admin-approved spender registration and authorization checks.
- [x] Implement positive-amount reward distribution from the pool to a learner with events.
- [x] Implement donor funding, pause enforcement, and admin emergency sweep with tests.
- [ ] Support explicit asset configuration policy—single immutable asset per pool or safe multi-asset pools—with issuer/network validation.
- [ ] Add query methods for configured asset, balance/accounting totals, pause state, spender state, and limits.
- [ ] Add spender removal, per-spender allowance/budget, per-transaction cap, per-period cap, expiry, and purpose/program binding.
- [ ] Add idempotent payout IDs so repeated course, referral, scholarship, or quest calls cannot pay twice.
- [ ] Track total funded, reserved, distributed, refunded, and recovered amounts with conservation invariants.
- [ ] Restrict emergency sweep with pause, delay/multisig, destination allowlist, reason/event, and protection for reserved obligations.

### Reward policy and program funding

- [ ] Add a policy/config contract or registry entries for reward amount, asset/pool, effective version, caps, and funding program.
- [ ] Bind course completion rewards to a policy frozen with the course version.
- [ ] Add program budget creation, fund, reserve, release, refund, close, and remaining-balance queries.
- [ ] Prevent one course/program/spender from consuming funds reserved for another.
- [ ] Emit indexer-friendly events for policy change, budget state, reservation, payout, failure/release, and refund.

### Referral rewards

- [ ] Decide and document what referral state must be on-chain versus API-verified.
- [ ] If paid on-chain, add referral reward purpose/claim ID, qualified learner commitment, one-time payout, cap, and fraud-reversal policy.
- [ ] Ensure referral payout cannot be forged through direct Reward Pool calls or duplicated across accounts/wallet migrations.

### Scholarships

- [ ] Add scholarship/program configuration for sponsor, eligibility commitment, cohort, milestones, reward schedule, start/end, capacity, and refund rules.
- [ ] Add milestone attestation/claim with verifier authorization and replay protection.
- [ ] Add program pause/cancel, learner removal where legally allowed, dispute window, unclaimed-fund recovery, and closeout.
- [ ] Add tests for exhausted budgets, concurrent claims, late claims, cancelled programs, verifier rotation, refunds, and accounting invariants.

**Closure evidence:** every treasury movement is authorized, capped, idempotent, attributable to a funded policy/program, and conserved under invariant tests.

## Phase 4 — Quests and employer-funded work

**Goal:** make Build and Explore quests safe, fully specified employer/partner incentive products.

### Quest lifecycle

- [x] Implement employer-funded Build Quest creation and quest-created events.
- [x] Implement learner proof-hash submission and duplicate-submission rejection.
- [x] Implement employer approve/reject review, batch approval, fee-adjusted payout, and review events.
- [x] Implement employer refund for inactive/unawarded quest state covered by current tests.
- [x] Implement admin-created Explore Quests and admin verification through Reward Pool.
- [x] Implement Quest Engine pause and upgrade entry points.
- [ ] Add quest title/metadata commitment, requirements/schema, review authority, capacity, per-winner reward, total escrow, start/deadline/review/refund windows, and status.
- [ ] Separate Build Quest employer review from Explore Quest trusted verifier/oracle policy.
- [ ] Add explicit quest activation, closure, cancellation, expiration, winner limit, and remaining-escrow accounting.
- [ ] Prevent proof submission before start/after deadline, more submissions than allowed, duplicate proofs/learners, and reviews after closure/refund.
- [ ] Add partial payout and multi-winner accounting without exceeding escrow.

### Proof, review, and dispute

- [ ] Domain-separate proof commitments by quest, learner, submission, schema, and version.
- [ ] Add review reason commitment, reviewer identity, reviewed timestamp, and immutable status transition rules.
- [ ] Add rejection appeal/dispute window and authorized resolution or document why disputes remain off-chain.
- [ ] Make batch review bounded, idempotent, and safe when one entry is invalid.
- [ ] Route payouts through policy-controlled Reward Pool/escrow consistently rather than mixing direct token transfers and pool transfers without a documented accounting model.
- [ ] Add cancellation/refund protections when accepted submissions or reserved payouts exist.

### Verification and integration

- [ ] Add conservation invariants for employer deposit, platform fee, learner payout, reserved balance, and refund.
- [ ] Add adversarial tests for duplicate review, review/refund race, malicious employer/reviewer, fee overflow, multiplier overflow, insufficient escrow, pause, and dependency failure.
- [ ] Deploy a complete testnet quest with API proof review and app status reconciliation.

**Closure evidence:** multiple learners can submit, be reviewed/disputed, and be paid from bounded employer escrow with no double payout or fund loss.

## Phase 5 — Stake Vault and reward multipliers

**Goal:** provide optional staking incentives without trapping funds or destabilizing reward accounting.

- [x] Implement authenticated staking, accumulated balance, lock timestamp reset, and stake events.
- [x] Implement lock-enforced full unstake and prevent repeated withdrawal.
- [x] Implement fixed-tier multiplier queries and upgrade entry point.
- [ ] Specify staking purpose, token, minimum/maximum, lock policy, tier thresholds, multiplier units, rounding, and who funds boosted rewards.
- [ ] Decide whether adding stake resets the whole lock, creates tranches, or uses weighted locks; implement and document the selected behavior.
- [ ] Add partial unstake or explicitly justify full-only unstake in product/economic review.
- [ ] Add user position query with principal, unlock time, multiplier, and pending state.
- [ ] Add pause/emergency behavior that preserves user withdrawal rights and cannot sweep user principal.
- [ ] Add total-staked accounting and token-balance conservation invariants.
- [ ] Version multiplier policy so existing locked positions cannot be changed unfairly without defined migration/notice.
- [~] Quest Engine currently consumes Stake Vault multipliers and caps payout; funding and fee conservation still require explicit refactoring.
- [ ] Define whether course rewards also receive multipliers and bind that rule to reward policy.
- [ ] Test lock boundaries, repeated deposits, partial/full withdrawal, policy upgrade, pause, malicious token, overflow/rounding, and boost exhaustion.

**Closure evidence:** each user can independently verify principal, unlock time, and multiplier; all stake is withdrawable under policy and boosted rewards remain fully funded.

## Phase 6 — Governance

**Goal:** deliver a complete, bounded governance lifecycle rather than voting on pre-seeded storage.

- [x] Implement badge-count-weighted voting and duplicate-vote prevention for stored proposals.
- [x] Implement proposer/admin cancellation with lifecycle checks and cancellation events.
- [x] Implement pass/fail/tie/active checks and execution marking/events for stored proposals.
- [x] Implement Governance upgrade authorization and negative tests.
- [ ] Add an authorized/public `create_proposal` entry point with proposer rules, action type, target/parameters commitment, metadata, voting start/end, quorum, threshold, and unique ID.
- [ ] Snapshot voting eligibility/weight at a defined ledger so badges acquired or revoked mid-vote cannot unpredictably alter completed votes.
- [ ] Define one-person, one-badge, or weighted-badge rules and resistance to wallet splitting/duplicate credentials.
- [ ] Add proposal states and queries for pending, active, succeeded, defeated, cancelled, queued, executed, and expired.
- [ ] Add quorum and approval threshold calculation with checked arithmetic and tie behavior.
- [ ] Restrict cancellation by lifecycle and define proposer/admin emergency authority.
- [ ] Replace “mark executed” with timelocked, allowlisted execution for approved protocol actions or explicitly scope governance to signaling only.
- [ ] Add execution replay protection, target validation, failure/retry semantics, and execution result events.
- [ ] Test proposal spam, double voting, badge transfer/revocation edge cases, flash staking if used, quorum boundary, cancellation race, execution replay, and malicious targets.

**Closure evidence:** proposals can be created, voted, finalized, queued, and safely executed—or are explicitly documented and enforced as non-binding signaling.

## Phase 7 — Privacy-preserving skill proofs

**Goal:** allow learners to prove selected claims without revealing their full credential history.

- [ ] Define claim schemas for course/path/skill completion, score bands, issuer, recency, and credential status.
- [ ] Select a Stellar-compatible commitment and ZK/selective-disclosure architecture after cryptographic review.
- [ ] Bind proofs to credential ID/status, holder, verifier challenge, domain, nonce, and expiry.
- [ ] Add proof-verifier contract or verified off-chain design with on-chain status roots/commitments as appropriate.
- [ ] Add issuer/status root rotation, revocation propagation, schema versioning, and backward compatibility.
- [ ] Prevent replay, correlation beyond disclosed claims, malicious verifier challenges, stale status proofs, and cross-domain proof reuse.
- [ ] Benchmark proof generation on target low-end mobile hardware and verification cost on Stellar.
- [ ] Complete independent cryptographic audit and API/app interoperability tests.

**Closure evidence:** a verifier can validate a narrowly disclosed, fresh, non-replayable claim while undisclosed learner achievements remain private.

## Phase 8 — Audit, testnet, mainnet, and protocol operations

**Goal:** deploy and operate the complete protocol safely at the TRD’s target scale.

### Verification and audit

- [ ] Add property, fuzz, invariant, differential, storage-migration, and cross-contract failure tests for all value and credential paths.
- [ ] Benchmark Wasm size, CPU/instruction use, storage growth, event size, batch limits, and transaction cost.
- [ ] Complete independent contract, economic, key-management, and privacy reviews and close every critical/high finding.
- [ ] Freeze versioned interfaces and publish source-to-Wasm provenance, hashes, specs, addresses, and known limitations.

### Testnet release

- [ ] Deploy all contracts in dependency order with reproducible configuration and documented addresses.
- [ ] Configure roles, multisig/timelock, spenders, assets, policies, budgets, and indexer cursors.
- [ ] Run app/API testnet journeys for course reward/credential, referral, scholarship, quest, stake, governance, pause, upgrade, and recovery.
- [ ] Rehearse underfunding, paused dependency, failed payout, TTL expiry, key rotation, upgrade, rollback, and event replay.

### Mainnet and ongoing operations

- [ ] Complete canary deployment/transactions before enabling public funding and rewards.
- [ ] Add monitoring for pool/program balances, payout anomalies, credential/revocation events, pauses, role/config changes, TTL, and upgrades.
- [ ] Establish treasury minimums, liabilities, reconciliation cadence, incident response, responsible disclosure, and recurring audit policy.
- [ ] Validate capacity/cost for 200,000 completions and 30,000 credentials with documented scaling limits.
- [ ] Complete staged mainnet enablement and post-launch protocol review.

**Closure evidence:** the audited protocol is reproducibly deployed, monitored, reconciled, recoverable, and every item in this roadmap is `[x]`.
