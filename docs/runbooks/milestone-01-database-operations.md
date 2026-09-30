# Milestone 01 Database Operations Plan

Status: planning runbook. Commands and automation described here will be implemented only after authorization; no database has been provisioned or migrated by this planning work.

Related: [Milestone plan](../milestone-01.md), [database ownership ADR](../adr/003-service-database-ownership.md), [migration ADR](../adr/005-database-migrations.md).

## 1. Environments and ownership

Use an explicitly named local development environment and disposable test containers. Each owns synthetic data only. Runtime services connect only to their logical database; separate migration credentials own schema changes. Bootstrap credentials create the three databases/roles, then are removed from service configuration.

| Database | Runtime table permissions |
| --- | --- |
| Product | Read/insert/update products; archive through update; no physical delete or DDL |
| Inventory | Read/insert/update stock; read/insert/update reservation decisions; read/insert reservation lines; no runtime history deletion or DDL |
| Order | Read/insert/update orders; read/insert immutable order lines; no runtime line mutation/history deletion or DDL |

Read-only migration history access is allowed for readiness. Revoke broad CONNECT/schema-create grants where needed and grant only necessary schema usage/table permissions. New tables need explicit grants through the owning migration procedure.

Test cleanup runs under disposable-environment administration credentials, not runtime roles. A stop task preserves volumes. Destructive reset is a distinct opt-in task with explicit environment selection.

## 2. Initial setup order

1. Record approved version pins, Docker platform support, local ports, and actual host tool versions.
2. Start the selected PostgreSQL image with a persistent development volume.
3. Bootstrap `product_db`, `inventory_db`, and `order_db`, each with separate migration/runtime roles and restricted grants.
4. Apply Product, Inventory, and Order migration histories through controlled owning-service runners. No cross-database FK imposes a DDL order, but keep this order for a readable setup procedure.
5. Verify migration status, schema constraints, and runtime permissions before service readiness succeeds.
6. Start Product, Inventory, Order, Gateway, and frontend. Order recovery starts only after its own schema is ready.
7. Add deterministic synthetic catalog and stock through owning APIs and generate local identity fixtures. Schema migrations do not create business seed data.
8. Record the first customer flow and stock/order invariants.

Use native Node environment loading and a standalone compiled DataSource. Separate source and compiled artifacts; never discover duplicate entities/migrations from stale output. Native UUID generation needs no PostgreSQL UUID extension or runtime extension-install permission.

## 3. Migration preparation and execution

### Prepare

- Define the business/schema change and owning service. A field for a future milestone is not added speculatively.
- Specify old/new application compatibility, expected data impact, DDL lock level, transaction behavior, and forward-repair/restore options.
- Generate or author a migration, then review its SQL. Generation is not evidence that the migration is safe or useful.
- Rehearse on an empty database and a populated disposable copy. Capture duration, row counts, and application/invariant checks.

### Execute

Use a compiled controlled migration runner rather than implicit service startup execution. Its standalone TypeORM DataSource creates a QueryRunner; the runner holds a service/database-specific session advisory lock on the same connection supplied to the migration executor. TypeORM 1.1.1's executor accepts an explicit QueryRunner. Timeout competing runners, and release the lock/connection in cleanup. Validate the lock lifetime and failure cleanup during implementation Stage A. [Versioned executor source](https://github.com/typeorm/typeorm/blob/1.1.1/src/migration/MigrationExecutor.ts)

Set finite lock/statement limits appropriate to the reviewed operation. Initial simple migrations use transactional DDL. Operations that cannot run within a transaction require an explicit reviewed exception and recovery procedure; no such advanced operation is needed for the initial schema.

Inspect migration history before and after. Apply only pending migrations. Record the target environment/database, application version, applied migration identifiers, duration, and result. Applied migrations are immutable; unexpected schema drift is diagnosed and corrected with a new migration.

Grant runtime permissions for new tables explicitly. Verify readiness can read required migration identifiers but cannot execute DDL. Readiness recognizes required applied migrations, tolerates compatible additive migrations, and fails for missing required history. Later destructive changes require an explicit compatibility boundary; merely counting migration rows does not prove compatibility.

### Failure

If transactional DDL fails, verify rollback and history before retrying. Do not blindly rerun partially executed nontransactional changes. For data-changing failures, use the reviewed repair or restore procedure. Automatically running `down` is not the default recovery action.

Application rollback may leave an additive schema intact if the older application remains compatible. A schema reversal and an application rollback are separate operations with separate evidence.

## 4. Required additive-migration experiment

Use a disposable populated copy. Choose a useful nullable descriptive field in the owning service during the exercise; it is not a predesigned future schema dependency.

Measure baseline data and behavior. Add the field through a migration. Verify fresh creation, populated upgrade, existing rows, stock/reservation totals, immutable order amounts, and old application behavior. Repeat the migration task and confirm it finds no pending copy of the same migration.

Record observed lock duration and the difference between migration success and application compatibility. Backfill and later constraint tightening/removal are Milestone 02 exercises.

## 5. Backup and restore rehearsal

### Establish the backup point

Stop accepting public writes and pause Order reconciliation. Drain in-flight Product, Inventory, and Order writes; ensure all transactions finish. Pending orders may remain, but their durable state is captured consistently and can recover after restore.

Capture logical dumps for all three databases and a safe role/grant reconstruction manifest. Keep credentials/private token keys outside committed artifacts. Record schema history, row counts, and invariant checks at the quiesced point. Resume writes/recovery only after the capture completes.

### Restore independently

Restore into a separate PostgreSQL instance/volume. Recreate roles and schema ownership, load all three databases, and reapply required privileges. Do not overwrite the original development instance as the first test.

Check constraints, migration identifiers, catalog references, reservation decisions/lines, order snapshots/totals, and table permissions. Restart the services against the restored instance, reconcile pending work, and run the customer flow. Check reserved quantities against successful reservation lines and confirmed/rejected orders against their Inventory decisions.

Record backup duration, restore duration, reconciliation duration, and data verification. A backup file existing is not proof of restore success. This exercise has a maintenance window; it does not claim live multi-database consistency or point-in-time recovery.

## 6. Connection and query baseline

Proposed local runtime pools start at five connections per business-service instance: three services use at most 15 application connections; Gateway has none. Migration runners use a bounded single connection per target. Reserve capacity for bootstrap/admin, probes, and experiments; measure actual pool behavior rather than assuming every configuration creates the same count.

Document PostgreSQL's configured connection limit. Include every replica and process when calculating the budget; test containers have separate budgets from development. Pool waiting counts toward the request deadline.

Capture plans for active-product pagination, customer order pagination, and due-pending-order lookup on deterministic synthetic datasets. Record distribution, indexes, execution time, and machine resources. Measure stock-row contention separately from read-query tuning.

PgBouncer, replicas, partitioning, and sharding remain later learning exercises. Increasing frontend/API replicas cannot independently increase primary inventory-write capacity.

## 7. Pending-order recovery procedure

Inspect safe order IDs/statuses and recovery timing in Order. Inspect Inventory's decision through its authenticated API using the same order ID. If absent, repeat the saved reservation operation idempotently; if present, apply its terminal outcome conditionally to the pending order.

Do not create a replacement order, change quantities/prices, fabricate rejection, or directly edit reserved counts to clear uncertainty. A permanent dependency outage leaves pending work visible until the dependency is repaired. No timeout-based release operation exists in Milestone 01.

Record the recovery result and recheck the matching stock invariant. The automated reconciler and any manual repair path use the same operation semantics.

## 8. Evidence to retain

Record environment/versions, migration identifiers, permission checks, backup/restore timing, row/invariant comparisons, query plans, concurrency outcomes, and safe recovery IDs. Exclude private keys, credentials, token fixtures, database URLs, and large raw dumps from Git.

Use the [experiment template](../experiments/TEMPLATE.md). Implementation evidence is added only after the operations have actually run.
