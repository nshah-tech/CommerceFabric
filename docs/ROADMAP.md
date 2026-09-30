# CommerceFabric Roadmap

Status: Milestone 1 planning package complete and ready for final owner review. Its overall design and version policy were accepted on 2026-09-30. Implementation has not started.

This roadmap translates the [master plan](../CommerceFabric_Master_Plan.md) into 15 milestones with bounded scope and reviewable completion criteria. CommerceFabric is a learning project. Each milestone should produce a working demonstration, test evidence, and an explanation of the decisions and failures encountered.

## Planning rules

- This document defines the overall sequence. [Milestone 1](milestone-01.md) contains the current detailed design, supported by the [Domain Model](domain-model.md).
- Design a milestone's database only when planning that milestone. Future tables, relationships, and indexes are deliberately unspecified here.
- Complete and review the active milestone's design before authorizing its implementation. A planning document is not implementation approval.
- Introduce technology to solve a demonstrated problem. Record the baseline before comparing alternatives.
- Keep database ownership, contracts, testing, security, and observability consistent as services evolve.
- Progress locally first. Cloud deployment is optional and has a separate cost and teardown plan.
- Completion requires observable behavior and evidence, not just installing a tool.

## Sequence and dependencies

| Milestone | Goal | Prerequisites | Master-plan phases |
| --- | --- | --- | --- |
| 01 | Microservices foundation | Planning review and implementation authorization | 0–4 |
| 02 | Distributed communication and safe API evolution | 01 | 5; extends initial order behavior |
| 03 | Real-time updates across instances | 02 | 6–8 |
| 04 | Reliable background processing | 03 | 9–10 |
| 05 | Durable business events and replay | 04 | 11–12 |
| 06 | Event correctness and transactional outbox | 05 | 13 |
| 07 | Containers, orchestration, and connection scaling | 06 | 14–16 |
| 08 | Full observability | 07 | 17–19 |
| 09 | Reproducible infrastructure and database relocation | 08 | 20–21 |
| 10 | Polyglot services with stable contracts | 09 | 22–24, 45 |
| 11 | Recommendation systems and data foundations | 10 | 25–26, selected 28, 33 |
| 12 | Business forecasting and decisions | 11 | 27 |
| 13 | Customer AI grounded in tools and retrieval | 12 | 28–33, applicable 40–41 |
| 14 | Evidence-based AIOps | 13 and usable telemetry from 08 | 34–39, applicable 40–41 |
| 15 | Production simulation and advanced database operations | 14 | 42–44, 46–47 |

This is the default learning order, not a deployment dependency graph. For example, the eventual Recommendation Service need not call Business AI. Phase references identify the master-plan material covered, rather than creating a second phase numbering scheme.

Deliberate sequencing adjustments: database containers, minimal frontend, request idempotency, basic authorization, logging, and initial CI begin in 01. Notifications are introduced in 03. Event deduplication and outbox remain in 06. Complete delivery automation remains in 15. AI evaluation starts with each AI capability.

## Common completion criteria

Every milestone must include:

1. Reviewed scope, contracts, and any schema changes required by that milestone.
2. A repeatable demonstration and relevant automated checks.
3. At least one documented failure or negative scenario.
4. Measured evidence where a performance or quality claim is made.
5. Architecture decisions and an experiment record describing results, limitations, and lessons.
6. Updated setup instructions and known unfinished work.

## Milestone scopes

### 01 — Microservices foundation

**Goal:** Understand service boundaries, database ownership, transactions, and concurrency through one complete commerce flow.

**Scope:** TypeScript/NestJS Gateway, Product, Inventory, and Order services; a minimal React frontend; REST/OpenAPI; three logically separate PostgreSQL databases; product administration; stock administration; order creation and retrieval; stock reservations; customer isolation; request idempotency; recovery of pending orders; migrations; initial CI and test environments.

**Deliverables:** The detailed design in [milestone-01.md](milestone-01.md), service and API diagrams, current database model, migration procedures, planned repository layout, implementation checklist, tests, and experiment notes.

**Exit evidence:** A customer can browse and order; two orders competing for the last item yield one confirmation; duplicate submissions have one stock effect; a lost reservation response recovers; customer isolation holds; a populated database survives a schema migration and backup/restore exercise.

**Excluded:** Payments, shipping, order cancellation, reservation expiry, brokers, gRPC, Kubernetes, and AI. These boundaries keep the first flow explainable; later milestones expand behavior explicitly.

### 02 — Distributed communication and safe API evolution

**Goal:** Compare synchronous REST and gRPC and evolve a deployed contract safely.

**Scope:** Protobuf contracts for selected Order-to-Inventory operations, deadlines, bounded retries with backoff, error mapping, API version compatibility, service authentication, and comparison against the REST baseline. Extend the order design with cancellation and a coordinated reservation-expiry protocol before implementing either.

**Database learning:** An expand → backfill → switch → contract schema exercise. Test old and new application versions against the expanded schema. Measure DDL lock behavior; rehearse interruption and resumable backfill. Design new reservation states only in this milestone's plan.

**Deliverables:** REST/gRPC comparison, compatibility tests, timeout policy, cancellation/expiry state machines, and schema rollout runbook.

**Exit evidence:** Both transports pass the same behavior tests; timeouts do not create extra reservations; cancellation releases once; expiry cannot invalidate an already confirmed allocation; old and new versions coexist during the migration exercise.

**Excluded:** Event-driven orchestration, message brokers, and language migration.

### 03 — Real-time updates

**Goal:** Deliver inventory and order changes to connected customers across multiple instances.

**Scope:** TypeScript Notification Service, authenticated WebSockets, customer-scoped subscriptions, Redis Pub/Sub, reconnect handling, browser tests, and snapshot refresh after reconnect.

**Deliverables:** WebSocket contracts, authorization rules, multi-instance topology, reconnection policy, and delivery experiments.

**Exit evidence:** Active subscribers receive relevant updates across instances; customers cannot subscribe to another customer's orders; reconnect retrieves current state; Redis interruption has documented degradation and recovery.

**Excluded:** Durable WebSocket history or guarantees that Redis Pub/Sub replays missed messages. Durable business events arrive in 05.

### 04 — Reliable background processing

**Goal:** Understand queue-based work, delivery acknowledgments, and recovery.

**Scope:** RabbitMQ, worker processes, email abstraction with a local capture sink, routing, ACK/NACK, prefetch, bounded retries, poison messages, dead-letter queues, and queue-pressure experiments.

**Deliverables:** Work contracts, retry and DLQ policy, worker tests, and operator replay procedure.

**Exit evidence:** Worker crash causes redelivery; retry limits hold; poison messages reach a DLQ; duplicate work avoids duplicate observable effects within the documented adapter boundary; backlog and processing latency are measured.

**Excluded:** Real email credentials, Kafka, and claims of exactly-once external delivery. Document the database-to-broker publication gap for 06.

### 05 — Durable business events

**Goal:** Understand event retention, consumer groups, ordering, and replay.

**Scope:** Kafka events for existing commerce behavior, AsyncAPI/JSON Schema contracts, explicit event IDs and versions, partition keys, consumer groups, lag, restart, and replay experiments. Add Schema Registry only when schema compatibility exercises need it.

**Deliverables:** Event catalog, retention/replay policy, event consumers, compatibility tests, and failure records.

**Exit evidence:** Consumers resume from committed offsets; a new group replays history; per-key ordering assumptions are tested; duplicates are visible and their effects documented.

**Excluded:** Assuming database updates and publication are atomic. Outbox implementation follows in 06.

### 06 — Correctness and transactional outbox

**Goal:** Make event effects resilient to duplicate delivery and publisher failure.

**Scope:** Service-owned outbox data, atomic business-write/outbox transactions, publishers, durable consumer deduplication, retry behavior, ordering, and retention. Review synchronous recovery from 01 against the event-based design before changing orchestration.

**Database learning:** Migrate populated databases to support event publication and deduplication; define backfill boundaries and avoid publishing historical actions twice. Learn batch sizes, lock duration, and cleanup impacts.

**Deliverables:** Outbox/deduplication schema designs, event rollout plan, repair/replay procedures, and guarded database integration tests.

**Exit evidence:** Repeating an event 100 times has one business effect; a failed publisher leaves recoverable work; restart eventually publishes; replay and retention do not silently defeat deduplication.

**Excluded:** A distributed transaction spanning PostgreSQL and Kafka or unconditional exactly-once delivery claims.

### 07 — Containers and orchestration

**Goal:** Run and scale application processes independently while respecting database limits.

**Scope:** Full service containers, complete Compose stack, local Kind/k3d Kubernetes, Helm, probes, resources, configuration, secrets, rolling updates, and independent API/worker replica counts.

**Database learning:** Separate application replica scaling from database capacity. Calculate aggregate connection budgets; compare bounded per-instance pools with PgBouncer; verify pool-mode compatibility. Treat PostgreSQL initially as one primary with persistent storage, distinct from stateless deployments.

**Deliverables:** Deployment configuration, pool sizing experiment, migration job ordering, and storage/restart runbooks.

**Exit evidence:** Rolling updates pass smoke tests; increasing replicas preserves correctness; connection counts remain bounded; PostgreSQL restart preserves data; migrations run once through a controlled job rather than each application replica.

**Excluded:** Multi-primary databases, production Kubernetes database operators, and sharding.

### 08 — Full observability

**Goal:** Explain behavior and failures using metrics, logs, and traces.

**Scope:** OpenTelemetry Collector, trace backend, Prometheus, Grafana, Loki, Alertmanager, correlation across HTTP/gRPC/messages, service and business dashboards, and alert exercises.

**Database learning:** Observe slow queries, query plans, lock waits, connections, deadlocks, and maintenance behavior. Use workload evidence to revise indexes and compare results before and after tuning.

**Deliverables:** Telemetry conventions, dashboards, alert rules, initial service-level objectives, and database tuning experiments.

**Exit evidence:** An order is traceable across services; a controlled slow query is diagnosable; alerts identify actionable symptoms; tuning improves a measured workload without breaking writes.

**Excluded:** Automated remediation and unmeasured performance claims.

### 09 — Infrastructure and database relocation

**Goal:** Recreate environments and move a service-owned database without losing ownership or data.

**Scope:** Terraform modules and environment boundaries, state management, local providers, infrastructure validation, destroy/recreate exercises, and an optional AWS deployment with budget and teardown instructions.

**Database learning:** Move one logical database from the shared local PostgreSQL instance to a dedicated instance using a planned maintenance window and dump/restore. Validate counts, constraints, references, application behavior, roles, and backups before cutover; retain a documented rollback path.

**Deliverables:** Infrastructure plans, relocation/cutover runbook, reconciliation results, and recovery timing.

**Exit evidence:** Environment recreation is repeatable; the relocated service works through the same API; reconciliation passes; rehearsed rollback restores the previous topology.

**Excluded:** Claiming zero downtime from dump/restore. An online migration comparison is reserved for 15.

### 10 — Polyglot engineering

**Goal:** Change implementation language while preserving behavior and data contracts.

**Scope:** Introduce a small Python/FastAPI service boundary and reimplement Inventory or Notification in Go; shared REST/gRPC/event tests; telemetry and authentication parity; race detection; profiling and benchmark comparison.

**Database learning:** Keep schema migration ownership with the service capability rather than maintaining competing TypeORM and Go migration histories. Test database compatibility before switching implementations.

**Deliverables:** Cross-language contracts, migration ownership decision, parity suite, and TypeScript/Go comparison.

**Exit evidence:** Both implementations pass the same contracts and concurrency tests; switch and rollback preserve data; measured comparisons use the same workload and environment.

**Excluded:** Rewriting every service and unrelated schema redesign.

### 11 — Recommendation systems

**Goal:** Learn how behavioral data becomes recommendations and how to measure quality.

**Scope:** Synthetic customer events, data quality, training/validation separation, rule baseline, content-based ranking, collaborative filtering, embeddings, pgvector, recommendation API, and MLflow.

**Database learning:** Design this milestone's feature/vector storage; compare vector-index choices and filtered retrieval using measured data. Preserve commerce database ownership.

**Deliverables:** Data pipeline, versioned evaluation dataset, baseline/model comparison, cold-start behavior, and Recommendation Service.

**Exit evidence:** Reproducible evaluation reports quality, coverage, diversity, cold starts, and latency; a model is compared against the baseline without training/test leakage.

**Excluded:** Live customer data, LLM shopping assistance, and business forecasts.

### 12 — Business AI

**Goal:** Distinguish prediction from a business decision.

**Scope:** Demand forecasting, seasonality, supplier lead-time assumptions, replenishment recommendations, simulated business outcomes, drift, and model tracking.

**Deliverables:** Business AI Service, baseline forecast, decision policy, evaluation data, and reports.

**Exit evidence:** Time-aware evaluation measures forecast error and bias; replenishment recommendations can be explained and compared in a simulation.

**Excluded:** Automated supplier purchasing and real financial decisions.

### 13 — Customer AI

**Goal:** Build assistance grounded in authorized APIs and retrievable sources.

**Scope:** Customer AI Service, product discovery, order lookup, policy RAG, pgvector retrieval, allowlisted tool calling, Langfuse, evaluation datasets, prompt-injection tests, and customer isolation.

**Deliverables:** Tool contracts, retrieval design, groundedness and tool evaluations, prompt/model version records, and usage/cost measurements.

**Exit evidence:** Expected tools and sources are used; unavailable data produces an explicit limitation; another customer's order is inaccessible; regression evaluations meet thresholds chosen before model comparison.

**Excluded:** Unrestricted tools, autonomous order placement, and invented operational facts.

### 14 — AIOps

**Goal:** Detect and diagnose operational incidents using cited evidence.

**Scope:** Statistical baselines, anomaly detection, temporal correlation, dependency graph, probable root causes, remediation recommendations, and human-approved actions. Progress to automation only after action-specific review, audit, verification, and rollback are designed.

**Deliverables:** AIOps Service, labeled incident dataset, operational evaluation, action catalog, and approval/rollback runbooks.

**Exit evidence:** Known incidents produce evidence-backed diagnoses; false-positive and root-cause metrics are reported; an approved action is verified; a failed action can be rolled back.

**Excluded:** Arbitrary shell execution, automatic destructive database actions, and autonomous schema migration.

### 15 — Production simulation and advanced database operations

**Goal:** Demonstrate system behavior under load, failure, recovery, and database change.

**Scope:** Full CI/CD, image scanning, deployment smoke tests, k6, targeted chaos, Python service hardening, and a hidden-failure simulation with diagnosis and recovery verification.

**Database learning:** Physical read replica and replication-lag experiments; primary-only consistency-sensitive reads; failover and write fencing; backup and point-in-time recovery; supported PostgreSQL major-version upgrade rehearsal; an online relocation experiment using logical replication; comparison with 09's maintenance-window migration. Partitioning is a measured lab on an appropriate accumulated dataset. Sharding is a trade-off assessment unless evidence warrants a separate milestone.

**Deliverables:** Capacity report, database operations runbooks, old/new compatibility evidence, cutover/rollback rehearsal, recovery objectives and measured results, and final learning portfolio.

**Exit evidence:** Load targets are agreed before testing; replicas demonstrate their lag and read-routing limits; recovery meets the chosen RPO/RTO; upgrade and relocation reconcile data; a hidden failure is diagnosed and resolved with verified recovery.

**Excluded:** A claim of production readiness or adding partitioning/sharding without a demonstrated benefit.

## Database learning progression

The term "migration" covers different operations. Each receives its own experiment and rollback/recovery plan.

| Skill | First milestone | Later depth | Evidence |
| --- | --- | --- | --- |
| Schema migrations | 01 | 02, 06 | Fresh install, populated upgrade, compatibility, preserved data |
| Data backfills | 02 | 06, 15 | Resumability, validation, bounded lock/transaction duration |
| Query and index tuning | 01 baseline | 08, 15 | Query plans, latency, write overhead, representative data |
| Backup and restore | 01 | 09, 15 | Successful restore and application checks, not just a backup file |
| Connection scaling | 07 | 15 | Aggregate connection budget, pool waits, throughput under replicas |
| Move a database to another host | 09 | 15 online comparison | Cutover, reconciliation, measured interruption, rollback |
| Read replicas and failover | 15 | Follow-up experiments | Lag, stale reads, fenced writers, recovery behavior |
| PostgreSQL major-version upgrade | 15 | Follow-up experiments | Compatibility checks, rehearsal, backup, recovery and reconciliation |
| Partitioning | 15 lab | Adopt only with evidence | Pruning and maintenance benefit against an unpartitioned baseline |
| Sharding | 15 assessment | Separate plan if justified | Constraints, routing, ownership, cross-shard costs |

Read replicas distribute suitable reads; they do not remove contention on the primary's inventory writes. More application replicas do not automatically mean more database capacity. These distinctions must be demonstrated in the scaling experiments.

Logical replication needs separate schema coordination and has limitations around sequences and other objects. An online cutover must account for these rather than treating replication as a complete database copy. [PostgreSQL logical replication restrictions](https://www.postgresql.org/docs/18/logical-replication-restrictions.html)

## Technology adoption

| Introduce | Technology | Reason |
| --- | --- | --- |
| 01 | Node.js, TypeScript, NestJS, React, Vite, PostgreSQL, TypeORM | Implement the first flow with explicit ownership and transactions |
| 01 | npm workspaces, Make, Docker Compose, Jest, Supertest, Testcontainers, Playwright, GitHub Actions | Repeatable local development and meaningful initial verification |
| 01 then 08 | Structured logs, OpenTelemetry, then the full monitoring stack | Inspect service interactions early and operate the growing platform later |
| 02 | gRPC, Protobuf; Buf when needed | Compare internal communication and manage contracts |
| 03 | WebSockets, Redis Pub/Sub | Fan out transient live updates across instances |
| 04 | RabbitMQ | Reliable background work |
| 05–06 | Kafka, AsyncAPI, schema tooling, outbox/deduplication | Durable events, replay, and reliable business effects |
| 07 | Kubernetes, Helm, PgBouncer | Orchestrate services and control database connections |
| 08 | Collector, Prometheus, Grafana, Loki, Alertmanager, Tempo or Jaeger | Metrics, logs, traces, and alerts |
| 09 | Terraform; AWS optionally | Reproducible infrastructure |
| 10 | Python, FastAPI, pytest, Go and its test/race/profiling tools | Cross-language engineering |
| 11–13 | pgvector, MLflow, Langfuse, model/LLM tooling | Evaluated recommendation, forecasting, and customer AI |
| 15 | k6, Trivy, selected chaos tools, PostgreSQL replication/recovery tools | Performance, delivery, and operational simulation |

Use latest stable runtime/library/tool versions when each milestone introduces them, checking compatibility before proceeding. Milestone 1's [matrix](technology/milestone-01.md) records the owner-approved TypeScript 6.0.3 exception; current Swagger/lint tooling excludes TypeScript 7. Pin selected versions and lockfiles during authorized setup. Future Python, Go, broker, and cloud versions are not chosen prematurely.

## Planning status and next action

- [x] Overall project goal documented in the master plan.
- [x] Milestone scope and database learning progression drafted here.
- [x] Milestone 1 design drafted in its detailed plan.
- [x] Owner accepts Milestone 1's overall scope and design baseline.
- [x] Complete its detailed API contracts, ADRs, database operations plan, and documentation consistency review.
- [x] Record exact versions, metadata compatibility, and the approved TypeScript exception.
- [ ] Owner completes final review of the detailed planning refinements.
- [ ] Authorize Milestone 1 implementation explicitly.

Only the first milestone has a detailed database model. Later milestone documents will be written when they become active.
