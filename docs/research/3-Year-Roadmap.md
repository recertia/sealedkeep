# Agentic Research Platform — 3-Year Roadmap (2026–2029)

**Document Status:** Authoritative  
**Version:** 1.0  
**Date:** 2026-10-27  
**Owner:** IT Architecture  
**Approval Model:** IT Architect decides; executive decisions are logged and final  

---

## Vision Restatement

By the end of 2029 the platform will be a production-grade, multi-tenant agentic research system that reliably externalizes iteration, lineage, control, and durable knowledge so that every important research output is fully traceable. Small teams will be able to run measured, reversible experiments at scale, coordinate agent swarms through a living commit DAG, enforce Software 3.0 control and evaluation outside any model context, and maintain a provenance-rich knowledge graph that agents query instead of rebuilding. The system will remain operable, recoverable, and evolvable by a small permanent team without rewriting its core invariants.

---

## Current-State Assessment

**Honest gaps (October 2026):**

- Phase 0 (Foundations) is complete: frozen OpenAPI contracts `v0.1.0`, JSON Schemas, Master Plan, and One-Pager exist.
- **No running code.** Phase 1 (Measured Loop) was authorized in July 2026 and has not started. There is no service skeleton, no CI pipeline, no Git repository with implementation, no authentication, no audit trail implementation.
- Team skill inventory, concrete technology choices (beyond logical planes), cost model, security review, and operational runbooks are missing.
- The architecture is specification-complete but implementation-zero. The largest risk is that the frozen contracts become outdated before any code is written against them.

**What is solid:**
- Clear four-layer model and five-plane reference architecture.
- Twelve non-negotiable invariants and acceptance tests.
- Explicit decision log and approval model.

---

## Target Architecture (End of 2029)

The five-plane architecture remains the stable core. By 2029 it will have matured as follows:

| Plane | 2029 Capability | Rationale |
|-------|-----------------|-----------|
| Control | Multi-tenant control documents, plan versioning, policy-as-code, automated termination and escalation | Enables organizational use without per-team reimplementation |
| Execution | Isolated, resource-quotaed runs (containers or micro-VMs), GPU scheduling for training loops, tool sandboxing with least privilege | Supports real ML workloads and concurrent agents safely |
| Artifact | Immutable object store + Git, content-addressable, retention policies, cross-region replication | Durable evidence for audit and recovery |
| Graph | Production property graph (or high-scale relational encoding) with compaction, semantic indexing, reversible entity resolution, and token-budgeted retrieval | Prevents context explosion and supports multi-hop reasoning |
| Evaluation | Hybrid deterministic + model + human gates, continuous metric monitoring, automated regression detection | Protects against metric gaming and silent quality decay |

Core invariants (especially full traceability and bounded context) remain unchanged. Technology choices stay conservative: Python for the majority of services, with Go or Rust only where binary size or concurrency density justifies the added cognitive load. All material choices will be recorded as ADRs.

---

## 3-Year Phased Roadmap

### Year 1 (2026–2027): Core Platform — From Spec to First Production Workload

**Goal:** Deliver a working measured loop + DAG + evaluation plane that a single research team can use daily, with full traceability on every run.

| Milestone | Target | Exit Criteria | Risks / Rollback |
|-----------|--------|---------------|------------------|
| Y1-Q1 | Phase 1–3 complete | Loop, DAG, Evaluation + Budget planes green on acceptance tests; first real research surface running autonomously | Contracts drift → freeze new minor version only after tests pass |
| Y1-Q2 | Knowledge Graph MVP | Graph acceptance tests green; agents retrieve bounded subgraphs instead of full history | Over-merging → mandatory reversible resolution + human review queue |
| Y1-Q3 | Integration & Hardening | E2E traceability under concurrent load; security review complete; runbooks published | Cost overrun → hard budget enforcement already in contracts |
| Y1-Q4 | Production Pilot | One external or internal research team running real experiments; keep-rate and cost-per-improvement measured | Adoption failure → pause new features, double down on operator experience |

**Key decisions for Year 1:** Storage for Graph Plane (Neo4j vs. relational + recursive queries), execution isolation mechanism, and first production GPU scheduling approach. Each recorded as ADR.

### Year 2 (2027–2028): Scale, Multi-Tenancy, and Operational Maturity

**Goal:** Support multiple independent research teams (or projects) without cross-contamination, with operational controls that a small team can sustain.

| Milestone | Target | Exit Criteria | Risks / Rollback |
|-----------|--------|---------------|------------------|
| Y2-Q1 | Multi-tenant isolation | Tenants have separate control documents, budgets, and graph partitions; no cross-tenant data leakage | Isolation bugs → feature flag off multi-tenancy |
| Y2-Q2 | DAG & Graph compaction | Automated summarization, archival, and semantic deduplication keep query latency stable as history grows | Loss of provenance → compaction is additive, never destructive |
| Y2-Q3 | Observability & FinOps | Dashboards for keep-rate, cost-per-kept-improvement, graph size, and recovery time; automated alerts | Metric gaming → secondary constraints enforced in Evaluation Plane |
| Y2-Q4 | Self-service onboarding | New research team can be onboarded in <1 day with templates and policy packs | Support burden → limit to 5 tenants until ops metrics stabilize |

**Architectural principle:** Prefer horizontal scaling and partitioning over vertical complexity. Every new capability must preserve the twelve invariants.

### Year 3 (2028–2029): Advanced Capabilities and Ecosystem

**Goal:** Enable higher-order agentic research (agents that improve their own control documents and evaluation harnesses under human oversight) and selective external collaboration.

| Milestone | Target | Exit Criteria | Risks / Rollback |
|-----------|--------|---------------|------------------|
| Y3-Q1 | Controlled self-modification | Agents can propose changes to `program.md` / control documents; changes require dual evaluation + human approval | Uncontrolled mutation → kill-switch and version pinning |
| Y3-Q2 | Cross-project knowledge reuse | Opt-in sharing of verified claims and successful lineages between tenants with provenance preserved | Privacy leakage → default deny + explicit consent |
| Y3-Q3 | Advanced evaluation | Statistical significance testing, adversarial evaluators, and automated metric discovery pilots | Overfitting to eval → retain human acceptance gates |
| Y3-Q4 | Ecosystem readiness | Stable public API subset, documentation, and at least one external integration or research collaboration | Scope creep → freeze public surface to current invariants |

**Exit condition for the roadmap:** By end of 2029 a small permanent team can operate the platform, onboard new research programs, and evolve the system without violating traceability or introducing irreversible technical debt.

---

## Risk Summary (3-Year Horizon)

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Implementation never starts | High (current) | Critical | Immediate Phase 1 authorization with named owners |
| Metric gaming at scale | Medium | High | Secondary constraints + Evaluation Plane from day one |
| Graph explosion / latency | Medium | High | Token budgets, compaction, and indexing in Year 2 |
| Entity-resolution contamination | Medium | High | Reversible merges + mandatory review for low confidence |
| Team skill / sustainability | Medium | High | Conservative stack, knowledge transfer, ADRs |
| Cost runaway | Medium | Medium | Hard budgets already in contracts; FinOps in Year 2 |

---

## Next Actions (Immediate)

1. **Technical Lead** — Create the implementation repository and service skeleton that enforces the frozen `v0.1.0` contracts and the twelve invariants. Target: skeleton + CI green within 2 weeks.
2. **IT Architect** — Publish ADRs for Graph Plane storage choice and execution isolation mechanism within 10 days of Phase 1 start.
3. **Executive** — Confirm resource allocation (team size and GPU access) for Year 1 so Phase 1 can begin without further architectural delay.
4. **Security** — Schedule the initial security review for the authentication and sandbox model before the first production pilot.

**Single most important action right now:**  
Authorize and staff Phase 1 implementation against the already-frozen contracts. The architecture is ready; the implementation is not.

---

**End of 3-Year Roadmap**  
This roadmap extends the July 2026 Master Plan into a multi-year operational trajectory while preserving the core invariants and four-layer architecture. All subsequent planning must reference this document and the frozen contracts.
