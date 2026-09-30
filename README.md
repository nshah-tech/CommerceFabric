# CommerceFabric

A learning project for building a polyglot, distributed commerce platform with real-time features and AI.

CommerceFabric uses product catalogs, inventory reservations, orders, and notifications as practical exercises in distributed systems. The long-term goal is to learn how to design, build, test, deploy, observe, scale, troubleshoot, and evolve independently deployable services across TypeScript, Python, and Go.

**Purpose:** This project is purely for learning and experimentation. Production-like environments and failure simulations are learning milestones.

**Current status:** Planning stage. This repository contains the master plan and project documentation; application services, infrastructure, and runnable development commands have yet to be implemented.

The [CommerceFabric Master Plan](CommerceFabric_Master_Plan.md) is the source of truth for the overall goal, detailed phases, experiments, and milestone checklists.

Planning uses two layers: the [Roadmap](docs/ROADMAP.md) defines all milestone scopes, dependencies, and completion criteria; the [Milestone 1 plan](docs/milestone-01.md) details the first flow, service boundaries, database model, contracts, technology choices, and tests. Database schemas are designed only for the active milestone. Database migrations, relocation, scaling, and recovery are explicit learning tracks throughout the roadmap. These documents are planning drafts; implementation awaits design review and authorization.

The [Domain Model](docs/domain-model.md) explains Milestone 1's entities, ownership, relationships, state transitions, and business invariants, with worked examples.

## Learning approach

Follow the progression:

> Problem → constraint → architecture → technology → failure → improvement

Introduce each technology when a concrete problem calls for it. Start with a small set of commerce services, then add communication patterns, reliability, operational tooling, and AI incrementally.

- Give each service ownership of its data, even when databases initially share one PostgreSQL server.
- Define stable REST, gRPC, and event contracts so implementations can change languages without changing behavior.
- Test throughout development, including concurrency, duplicate delivery, outages, and recovery.
- Record architecture decisions, trade-offs, experiments, and evidence of what works.
- Develop locally first; AWS is an optional later learning target.

## Planned architecture

The initial application will connect a React frontend to a TypeScript API Gateway and Product, Inventory, and Order services. Notification and Python AI services will follow as the project progresses.

| Component | Responsibility | Planned language |
| --- | --- | --- |
| API Gateway | External API boundary, authentication, and routing | TypeScript / NestJS |
| Product Service | Product catalog | TypeScript / NestJS |
| Inventory Service | Stock, reservations, and releases | TypeScript initially; Go migration later |
| Order Service | Order lifecycle | TypeScript / NestJS |
| Notification Service | Real-time notifications and email abstraction | TypeScript initially; Go migration later |
| Recommendation Service | Product recommendations | Python / FastAPI |
| Business AI Service | Demand forecasts and replenishment decisions | Python / FastAPI |
| Customer AI Service | Product discovery and customer assistance | Python / FastAPI |
| AIOps Service | Operational analysis and remediation | Python / FastAPI |

Services will own their databases and communicate through explicit APIs and events. Later language migration exercises will preserve API contracts, event schemas, tests, and telemetry conventions.

## Planned technology stack

| Area | Technologies and purpose |
| --- | --- |
| Applications | React, TypeScript / NestJS, Python / FastAPI, Go |
| APIs and contracts | REST / OpenAPI, gRPC / Protobuf, AsyncAPI, JSON Schema |
| Data | PostgreSQL, TypeORM, pgvector for vector search |
| Real-time delivery | WebSockets; Redis Pub/Sub for delivery across multiple instances |
| Background work | RabbitMQ, workers, retries, and dead-letter queues |
| Business events | Kafka for durable events, consumer groups, and replay |
| Reliability | Transactions, idempotent consumers, transactional outbox |
| Infrastructure | Docker Compose, Kubernetes, Helm, Terraform; AWS later |
| Observability | OpenTelemetry, Prometheus, Grafana, Loki, Alertmanager, Tempo / Jaeger |
| AI engineering | Recommendation models, forecasting, RAG, LLM tool calling, MLflow, Langfuse |
| Verification | Jest, pytest, go test, Testcontainers, Playwright, k6, Trivy, AI evaluation datasets |

These are roadmap technologies, introduced gradually as described in the master plan.

## Learning roadmap

The master plan defines phases 0–47, grouped into 15 milestones:

| Milestone | Focus |
| --- | --- |
| 1. Microservices foundation | Gateway, products, inventory, orders, data ownership, and initial tests |
| 2. Distributed communication | REST, gRPC, Protobuf, timeouts, retries, and contract tests |
| 3. Real time | WebSockets, Redis Pub/Sub, multiple replicas, and reconnection |
| 4. Async processing | RabbitMQ, workers, acknowledgments, retries, and dead-letter queues |
| 5. Event streaming | Kafka topics, partitions, consumer groups, replay, and lag |
| 6. Correctness | Idempotency, duplicate events, ordering, and transactional outbox |
| 7. Containers and orchestration | Docker, Kubernetes, Helm, scaling, and deployment checks |
| 8. Observability | Metrics, logs, distributed traces, alerts, and telemetry tests |
| 9. Infrastructure | Terraform and reproducible infrastructure; optional AWS experiments |
| 10. Polyglot engineering | Python and Go services with shared contracts and concurrency tests |
| 11. Recommendation AI | Rule baselines, content-based and collaborative models, embeddings, and evaluation |
| 12. Business AI | Demand forecasting, replenishment recommendations, and model tracking |
| 13. Customer AI | RAG, tool calling, product search, order assistance, and regression evaluations |
| 14. AIOps | Anomaly detection, incident correlation, root-cause analysis, and approved remediation |
| 15. Production simulation | Load, failures, diagnosis, recovery, and recovery verification |

Testing spans every milestone. CI should run tests from the initial service skeleton, with broader delivery automation added as the platform develops.

## Planned repository layout

The master plan recommends a monorepo. The following directories will be introduced during implementation:

```text
services/         Commerce, gateway, notification, and AI services
contracts/        OpenAPI, AsyncAPI, and Protobuf definitions
frontend/         React application
ml/               Datasets, features, notebooks, models, and experiments
infrastructure/   Docker, Kubernetes, Helm, and Terraform configuration
monitoring/       Metrics, dashboards, logging, and alerting configuration
tests/            Integration, contract, E2E, load, chaos, and AI evaluations
docs/             Architecture decisions, experiments, AI documentation, and runbooks
scripts/          Development and automation helpers
```

## Getting started

1. Read the [master plan](CommerceFabric_Master_Plan.md), beginning with the architecture philosophy and Phase 0 development environment.
2. Establish Git, Node.js, Python, Go, Docker, Docker Compose, and Make as needed for the active phase.
3. Build the initial gateway and commerce service skeleton, with PostgreSQL integration tests and API contracts.
4. Progress through the plan, recording what each change solves, how it fails, and how its behavior is verified.

Phase 0 proposes `make dev`, `make test`, `make lint`, `make build`, and `make down`. These commands are planned; there is currently no Makefile or runnable application stack.

## Testing and experiments

Use the smallest test scope that proves the behavior: unit tests for business rules, integration tests for databases and brokers, contract tests across service boundaries, and selected end-to-end tests for customer flows.

Key experiments include reserving a single inventory item under concurrent requests, processing repeated events with one business effect, recovering outbox publication after a failure, and replacing a TypeScript service with Go while preserving its contracts.

AI features will use versioned evaluation datasets to measure recommendation quality, forecasting accuracy, retrieval quality, grounded answers, tool correctness, and customer data isolation. AIOps recommendations should cite operational evidence and begin with human approval; later automation will include restricted tools, audit logs, and rollback mechanisms.

## License

CommerceFabric is licensed under the [MIT License](LICENSE).
