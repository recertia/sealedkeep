# Agentic Research Platform — 3-Year Technical Implementation Plan and Specifications

**Document Status:** Authoritative  
**Version:** 1.0  
**Date:** 2026-10-27  
**Owner:** IT Architecture  
**Related:** MASTER-PLAN.md, 3-Year-Roadmap.md, Technical-Roadmap.md, contracts/v0.1.0  

---

## Vision Restatement

By end of 2029 the platform is a multi-tenant, production-grade agentic research system in which every important output is reconstructible to an objective, a plan, an artifact, a source, a graph path, an evaluator decision, and a bounded execution record. Iteration, lineage, control, and durable knowledge are first-class, explicit structures. A small permanent team can operate, onboard, and evolve the system without violating core invariants.

---

## Current-State Assessment

Frozen contracts `v0.1.0` exist. No implementation code, service skeleton, CI, authentication, storage, or ADRs exist. Phase 1 is authorized but unstarted. The gap between specification and running system is total. The highest near-term risk is that the frozen contracts age before any code validates against them.

---

## Target Architecture (End of 2029)

Five-plane architecture remains the stable core, matured as follows:

- **Control Plane**: Multi-tenant ControlDocuments, plan versioning, policy-as-code, automated termination.  
- **Execution Plane**: Resource-quotaed, isolated runs (containers or micro-VMs) with GPU scheduling for training loops.  
- **Artifact Plane**: Immutable Git + content-addressable object store with retention and replication.  
- **Graph Plane**: Production property graph (or high-scale relational) with compaction, semantic indexing, reversible entity resolution, and token-budgeted retrieval.  
- **Evaluation Plane**: Hybrid deterministic + model + human gates with continuous monitoring and regression detection.

Technology defaults remain Python (FastAPI) unless an NFR forces otherwise. All material choices are recorded as dated ADRs.

---

## Year-by-Year Technical Implementation Plan

### Year 1 (2026–2027): Core Platform

**Goal**: Single research team can run measured, reversible experiments with full traceability.

**Specifications**
- Control Plane implements `/control` and `/runs` exactly against frozen schemas. Postgres tables for ControlDocument, Run, Trial.
- Execution Plane: containerized workers (Docker/Podman or K8s Jobs) that execute the mutable surface under a hard time budget and return structured results.
- Artifact Plane: bare Git repository + MinIO (or S3-compatible). Every keep/revert is a Git commit; bundles are size-validated.
- Graph Plane (MVP): Postgres recursive CTEs (ADR required if Neo4j is chosen instead). Implements `/dag` and basic `/graph` subgraph retrieval under token budget.
- Evaluation Plane: `/eval` returns EvaluationResult with evidence_edge_ids; BudgetDeclaration is hard-enforced.
- Auth: per-agent Bearer API keys. Protected paths rejected before commit.
- Audit: append-only log that reconstructs objective → plan → runs → commits → claims → evaluations → budgets.

**Interfaces**: All public endpoints match `contracts/v0.1.0/openapi/agentic-platform-v0.1.0.yaml`. Internal worker contract is a simple JSON message (run_id, diff, timeout).

**NFRs**: 50-trial reproducibility; crash recovery to last kept commit; subgraph retrieval <2 s p95; hard budget enforcement.

**Exit criteria**: Loop, DAG, Evaluation, and Graph MVP acceptance tests green; one real research surface running autonomously.

**Risks / Rollback**: Contracts drift → only version after tests pass. Over-merging → reversible resolution + review queue from day one.

### Year 2 (2027–2028): Scale and Multi-Tenancy

**Goal**: Multiple independent teams operate without cross-contamination; a small ops team sustains the system.

**Specifications**
- Multi-tenant isolation: tenant_id on all ControlDocuments, Runs, and graph partitions. No cross-tenant reads by default.
- Compaction: additive summarization and archival jobs for DAG and graph (never destructive). Semantic deduplication for near-duplicate claims.
- Observability: Prometheus + Grafana (or equivalent) for keep-rate, cost-per-kept-improvement, graph size, recovery time, token usage.
- FinOps: cost attribution per tenant and per run; alerts on budget approach.
- Self-service: template ControlDocuments and policy packs; onboarding <1 day for new teams.
- Graph Plane: indexing and optional migration path to Neo4j if recursive CTE latency exceeds targets (ADR required).

**Interfaces**: Existing APIs remain stable. New admin endpoints for tenant management (versioned). Compaction jobs are internal.

**NFRs**: Query latency stable as history grows; zero cross-tenant leakage; recovery time objective <15 min for any plane.

**Exit criteria**: Multi-tenant isolation proven; compaction keeps latency stable; observability dashboards live; at least one additional team onboarded.

**Risks / Rollback**: Isolation bugs → feature-flag multi-tenancy off. Provenance loss → compaction is additive only.

### Year 3 (2028–2029): Advanced Capabilities and Ecosystem

**Goal**: Controlled higher-order agentic research and selective external collaboration under human oversight.

**Specifications**
- Controlled self-modification: agents may propose changes to ControlDocuments. Dual evaluation + human approval required. Kill-switch and version pinning.
- Cross-project reuse: opt-in sharing of verified claims and successful lineages with full provenance. Default deny.
- Advanced evaluation: statistical significance testing, adversarial evaluators, pilot automated metric discovery. Human gates retained.
- Public API subset: stable, documented subset of the frozen contracts plus selected Year 2/3 extensions. Versioned and rate-limited.
- Ecosystem: at least one external integration or research collaboration.

**Interfaces**: Public API is a strict subset of internal contracts. Self-modification proposals flow through the existing Evaluation API with an additional human-approval gate.

**NFRs**: Self-modification cannot bypass invariants; public surface does not expose internal graph or budget internals; human acceptance remains the final gate.

**Exit criteria**: Controlled self-modification demonstrated under dual evaluation; opt-in sharing works with provenance; public API documented and used by at least one external party.

**Risks / Rollback**: Uncontrolled mutation → kill-switch. Privacy leakage → default deny. Scope creep → freeze public surface to current invariants.

---

## Cross-Cutting Specifications

**Data models**: Remain those defined in the frozen JSON Schemas (ControlDocument, Trial, CommitNode, GraphUpdate, EvaluationResult, BudgetDeclaration, Run). Extensions in Years 2–3 are additive and versioned.

**Security guardrails (all years)**
- Per-agent API keys with least privilege.
- No shell access on control or graph services.
- Bundle size and rate limits.
- Protected evaluation surface never modified by agents.
- All writes carry provenance; entity resolution is reversible.
- Hard budget enforcement.
- Append-only audit log with defined retention.

**Integration patterns**
- Year 1: synchronous API calls + simple worker queue.
- Year 2: event-driven compaction and multi-tenant partitioning.
- Year 3: human-approval workflow for self-modification; opt-in export/import of verified subgraphs.

**Migration strategy**: All changes are additive. Existing contracts remain valid. New capabilities are behind feature flags until exit criteria are met. No big-bang rewrites.

**ADRs required**
- ADR-001: Graph Plane storage (Postgres vs Neo4j) — Year 1.
- ADR-002: Execution isolation mechanism — Year 1.
- ADR-003: Multi-tenant partitioning strategy — Year 2.
- ADR-004: Compaction and archival policy — Year 2.
- ADR-005: Self-modification approval workflow — Year 3.

---

## Quality Gates and Acceptance

Every phase must pass:
- Contract validation (requests and responses match frozen schemas).
- The twelve invariants (tested automatically).
- Phase-specific acceptance tests listed in the Master Plan and Technical Roadmap.
- Security review before any production pilot.
- Knowledge-transfer demo for the permanent team.

---

## Next Actions

1. **Technical Lead** — Initialize the implementation repository with contract-validating skeleton and CI (target: green CI within 2 weeks).
2. **IT Architect** — Publish ADR-001 and ADR-002 within 10 days of Phase 1 start.
3. **Developers** — Implement Year 1 Phase 1 only against frozen contracts until Loop acceptance tests are green.
4. **Executive** — Confirm Year 1 resource allocation (team + GPU access).

**Single most important action right now**  
Stand up the implementation repository and contract-enforcing CI. The 3-year technical plan cannot begin until code validates against the frozen contracts.

---

**End of 3-Year Technical Implementation Plan and Specifications**  
This document provides the technical specifications and sequenced implementation plan required to realize the 3-year vision while preserving the frozen contracts and core invariants.
