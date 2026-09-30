# ADR 005 — Schema migrations and database recovery

Date: 2026-09-30. Status: accepted Milestone 01 design baseline; implementation pending.

## Context

Schema evolution, data preservation, and database scaling are explicit learning goals. Automatic schema synchronization hides deployment order, lock effects, and data-change risks.

## Decision

Use versioned TypeORM migrations owned by each service capability, with auto-synchronization and startup migration execution disabled everywhere. Tests build schemas by applying the same migration history. Use a standalone explicit DataSource and a controlled compiled migration runner; use the CLI for generation/review. Load configuration explicitly rather than relying on removed legacy environment loaders.

One controlled runner per database holds a session advisory lock on the same connection used for migration execution. Define finite lock/statement limits and guaranteed cleanup. Run as the migration role; grant only required runtime table access afterward. Database bootstrap and business schema history are distinct tasks.

Review generated DDL before execution, retain immutable applied history, and validate schema compatibility during readiness. Native application-generated UUIDs need no extension installation; prevent automatic extension setup from requiring runtime DDL permissions.

Rehearse fresh creation, a populated additive schema change, old-application compatibility, repeat-run detection, and backup/restore to a separate instance. Capture roles/grants and verify aggregate stock/order invariants.

Stop writes and pause Order reconciliation before the first multi-database backup exercise; drain in-flight work and then capture the owned databases. Point-in-time recovery and online relocation come later.

## Alternatives

- ORM synchronization would obscure reviewed schema history and make data change effects difficult to reproduce.
- Migrating on every service startup would couple DDL permission and migration concurrency to application replicas.
- Automatically reverting schema on application rollback would risk discarding data unnecessarily.

## Consequences

Local setup includes a deliberate migration task. Additive schemas can remain during application rollback. Destructive reversals are rehearsed only on disposable copies; recovery may require a forward repair or verified restore. Language migration must preserve one schema history per owning capability.

## Verification

Migration-role separation, competing runner lock test, history checks, empty/populated upgrade, preserved business data, and a verified restore. [Database operations runbook](../runbooks/milestone-01-database-operations.md)
