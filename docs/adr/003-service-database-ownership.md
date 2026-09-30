# ADR 003 — Service-owned databases

Date: 2026-09-30. Status: accepted Milestone 01 design baseline; implementation pending.

## Context

The initial stack should be manageable on a local machine while teaching data ownership and independent migration. Separate machines are unnecessary for establishing the first ownership boundary.

## Decision

Use one PostgreSQL instance with `product_db`, `inventory_db`, and `order_db`. Each service receives its own application role restricted to its database and required DML. Separate migration roles own DDL. Bootstrap credentials do not belong in runtime configuration.

Remove inherited broad grants where required, restrict schema creation, and verify that each application role cannot access another database or perform DDL. Gateway has no database role.

Use foreign keys within each database only. Cross-service UUIDs are references resolved through APIs, not cross-database constraints or joins. Retain catalog identities and accepted order snapshots. No service loads another service's ORM entities.

Design only the current milestone's tables. Later schemas are introduced through owning-service migrations when the corresponding capability is planned.

## Alternatives

- One shared database with unrestricted tables would make direct cross-service writes easy and blur ownership.
- One database with separate schemas/roles can enforce boundaries, but separate logical databases make accidental coupling harder and support a clear relocation exercise.
- Three PostgreSQL servers would create more operational work before independent resource scaling is the learning goal.

## Consequences

The server remains a shared capacity and failure boundary. Cross-service referential integrity and recovery depend on contracts and retained identities. A logical database dump does not independently capture all server roles or a consistent multi-database recovery point during live writes.

Milestone 09 moves one owned database to another server while retaining its API identity and migration history. Scaling application replicas must respect connection limits; consistent stock allocation remains on the primary.

## Verification

Negative credential/DDL tests; current schema constraints; cross-service invariant checks; quiesced backup/restore with recreated roles. [Domain model](../domain-model.md), [migration runbook](../runbooks/milestone-01-database-operations.md)
