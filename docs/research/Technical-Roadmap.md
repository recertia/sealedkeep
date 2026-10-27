# Agentic Research Platform — Technical Roadmap

**Document Status:** Authoritative  
**Version:** 1.0  
**Date:** 2026-10-27  
**Owner:** IT Architecture  
**Scope:** Implementation sequence for the repository against frozen contracts v0.1.0  

---

## Vision Restatement

Deliver a production-grade agentic research platform whose every important output is reconstructible to an objective, a plan, an artifact, a source, a graph path, an evaluator decision, and a bounded execution record. The system externalizes iteration (measured loop), lineage (commit DAG), control (Software 3.0 documents + budgets), and durable knowledge (typed graph) so that agents scale capability without scaling opacity.

---

## Current-State Assessment

- Frozen OpenAPI contracts and JSON Schemas exist at `contracts/v0.1.0/`.
- No implementation repository, service skeleton, CI, authentication, audit trail, or storage exists.
- No concrete technology choices have been recorded as ADRs.
- Phase 1 (Measured Loop) remains authorized but unstarted.

The contracts are the single source of truth for interfaces. All implementation must validate against them.

---

## Target Technical Architecture

Five planes with clear data ownership and interface boundaries. Technology defaults to Python unless a non-functional requirement forces otherwise.

| Plane | Primary Responsibility | Data Ownership | Preferred Technology | Key Interfaces |
|-------|------------------------|----------------|----------------------|---------------|
| Control | Objectives, plans, budgets, termination | ControlDocument, Run | Python + FastAPI | `/control`, `/runs` |
| Execution | Isolated tool/training/sub-agent runs | Transient process state | Python + container runtime (Docker/Podman or Kubernetes Jobs) | Internal worker API |
| Artifact | Immutable blobs + Git history | Git repo + object store | Git + MinIO/S3-compatible | Git bundle endpoints, object URIs |
| Graph | Commit DAG + knowledge graph + links | CommitNode, Graph nodes/edges | PostgreSQL (recursive CTEs) or Neo4j; start with Postgres | `/dag`, `/graph` |
| Evaluation | Deterministic checks + model evaluators | EvaluationResult | Python (same process or separate service) | `/eval` |

**Architectural principles**
- Contracts first: no service may deviate from frozen OpenAPI/JSON Schema without a versioned ADR.
- Append-only where possible (ledger, Git, audit log, graph history).
- Bounded context construction only (token budget enforced at Graph Plane).
- Every write carries provenance (run_id, agent_id, timestamp).
- Isolation: agents never receive a shell on the control or graph services.

**Key non-functional requirements (measurable)**
- Traceability: 100% of important outputs reconstructible via audit API.
- Latency: subgraph retrieval under declared token budget in <2 s p95 for Year 1 workloads.
- Recovery: any run resumable from last kept commit or last successful graph transaction.
- Cost control: hard enforcement of BudgetDeclaration; no silent overruns.
- Operability: single-team operation; runbooks for the five failure modes listed in the Master Plan.

---

## Technical Delivery Sequence

### Phase 1 — Measured Loop (Weeks 1–6)
**Components:** Control Document store, Trial ledger, Git integration, evaluate/keep/revert cycle, single agent interface.
**Work packages**
- Service skeleton (FastAPI) that validates every request/response against frozen schemas.
- Postgres tables for ControlDocument, Trial, Run.
- Git wrapper that enforces protected paths and records every keep/revert.
- One LLM tool that emits unified diffs against the mutable surface.
**Exit criteria:** All Loop acceptance tests green (50-trial reproducibility, crash recovery, protected-path rejection).
**Risk / Rollback:** Agent proposes invalid edits → reject before commit (already in invariant).

### Phase 2 — DAG Collaboration (Weeks 5–10)
**Components:** Bare Git + bundle push/fetch, children/leaves/lineage/diff, message board.
**Work packages**
- Bundle validation, size/rate limits, per-agent API keys.
- Graph queries implemented as SQL (or Cypher if Neo4j chosen).
- Message-board tables and endpoints.
**Exit criteria:** Three concurrent agents produce correct multi-lineage DAG; all DAG tests green.
**Dependency:** Phase 1 ledger must be stable.

### Phase 3 — Evaluation Plane & Budgets (Weeks 9–12)
**Components:** Structured Evaluation API, BudgetDeclaration enforcement, full AuditTrail.
**Work packages**
- Evaluation service that returns pass/fail/revise with evidence edge IDs.
- Budget accounting middleware that aborts on exhaustion and returns structured partial result.
- Audit log that reconstructs the full chain.
**Exit criteria:** Traceability invariant holds for every run; budget exhaustion is clean.

### Phase 4 — Knowledge-Graph Overlay (Weeks 11–18)
**Components:** Node/edge schema, GraphUpdate path, bounded subgraph retrieval, reversible entity resolution.
**Work packages**
- Graph tables (or Neo4j schema) matching frozen GraphUpdate schema.
- Token-budgeted traversal (1–2 hops, prioritized verified claims).
- Resolution endpoint with confidence, rationale, and reversible merge records.
- Links from Commit and AgentRun nodes into the graph.
**Exit criteria:** All Graph acceptance tests green; context construction stays under budget.

### Phase 5 — Integration & Hardening (Weeks 17–22)
**Components:** Five-plane wiring, observability, failure injection, security review.
**Work packages**
- End-to-end scenarios that exercise the full traceability chain.
- Dashboards for keep-rate, cost, graph size, recovery time.
- Security review of API keys, sandboxing, and bundle validation.
- Runbooks for crash, budget exhaustion, and false-merge reversal.
**Exit criteria:** Full E2E + recovery tests pass under concurrent load.

### Phase 6 — Production Pilot (ongoing)
**Components:** Real research surface, operator experience, measurement.
**Exit criteria:** One team running real experiments; keep-rate and cost-per-improvement reported.

---

## Technology Decisions (to be recorded as ADRs)

| Decision | Recommended Default | Alternatives | When to Revisit |
|----------|---------------------|--------------|-----------------|
| API framework | FastAPI (Python) | Go (if binary size or concurrency density demands) | After first production load test |
| Primary store | PostgreSQL | Neo4j for Graph Plane | If recursive CTE latency exceeds targets |
| Execution isolation | Docker/Podman or K8s Jobs | Firecracker / gVisor | When GPU scheduling or multi-tenant isolation is required |
| Object store | MinIO or S3-compatible | Local filesystem (dev only) | Before production pilot |
| Auth | Per-agent API keys (Bearer) | OIDC / short-lived tokens | Year 2 multi-tenancy |
| Graph query | SQL recursive CTEs (start) | Cypher | If multi-hop performance degrades |

All choices must be documented in short, dated ADRs linked to the relevant phase.

---

## Data Ownership and Flow Boundaries

- Control Plane owns ControlDocument and Run lifecycle.
- Execution Plane owns only transient process state; results are written to Artifact and Graph planes.
- Artifact Plane owns Git objects and immutable blobs; no plane may mutate a kept commit.
- Graph Plane owns CommitNode, knowledge-graph nodes/edges, and the links between them.
- Evaluation Plane owns EvaluationResult; it reads from Graph and Artifact but does not write lineage.
- AuditTrail is append-only and owned by the Control Plane (or a dedicated audit service).

Cross-plane writes occur only through the published APIs or internal events that carry run_id and provenance.

---

## Security and Compliance Guardrails

- Every agent authenticates with a scoped API key.
- Agents never receive a shell on control or graph services.
- Bundle size and rate limits enforced at the DAG endpoint.
- Protected paths (evaluation surface) are rejected before commit.
- All graph writes include provenance; entity resolution is reversible.
- BudgetDeclaration is hard-enforced; exhaustion returns structured partial results.
- Audit log retains the full chain for the retention period defined in the ADR.

---

## Next Actions

1. **Technical Lead** — Initialize the implementation repository with the service skeleton, schema validation middleware, and CI that fails any contract deviation. Target: green CI on empty services within 2 weeks.
2. **IT Architect** — Publish ADR-001 (Graph Plane storage: Postgres recursive CTEs vs Neo4j) and ADR-002 (execution isolation) within 10 days of Phase 1 start.
3. **Developers** — Implement Phase 1 against the frozen contracts; no feature work outside the Loop acceptance tests until they are green.
4. **Security** — Review the API-key and sandbox model before the first production pilot.

**Single most important action right now**  
Stand up the implementation repository and contract-enforcing CI. Until that exists, the frozen contracts remain unused specifications.

---

**End of Technical Roadmap**  
This document translates the Master Plan and 3-Year Roadmap into sequenced technical work packages that honor the frozen contracts and twelve invariants. All implementation must reference it.
