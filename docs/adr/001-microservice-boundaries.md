# ADR 001 — Initial microservice boundaries

Date: 2026-09-30. Status: accepted Milestone 01 design baseline; implementation pending.

## Context

CommerceFabric is a learning project about independently owned and deployable services. Its first working flow needs catalog data, concurrent stock allocation, and a recoverable customer order. Language diversity, brokers, and infrastructure orchestration come later.

## Decision

Create four separate TypeScript/NestJS processes: API Gateway, Product, Inventory, and Order. Each has its own package, build/start/test commands, configuration, and health endpoints. Use a monorepo and npm workspaces for development convenience.

Product owns current catalog data and prices. Inventory owns stock and reservation decisions. Order owns customer orders, immutable price snapshots, submission identity, and outcome recovery. Gateway owns public routing and authentication, with no business persistence.

Services communicate through versioned HTTP contracts. Share small technical helpers only; do not share entities, repositories, database connections, or business orchestration. Contract documents stay language-neutral to support later Go/Python exercises.

## Alternatives

- A modular monolith would reduce network and recovery work, but would defer the service-boundary learning that is central to this project.
- More services for customers, pricing, carts, payments, or fulfillment would expand the first flow before those capabilities are needed.
- Starting with three languages would make failures harder to attribute before a stable behavior baseline exists.

## Consequences

Local setup needs four processes and service credentials. Partial failure and eventual convergence must be designed immediately. A monorepo does not imply shared persistence or one combined deployment artifact.

## Verification

Each service builds independently; APIs are explicit; runtime database access is isolated; accepted orders recover after a downstream interruption. [Milestone plan](../milestone-01.md)
