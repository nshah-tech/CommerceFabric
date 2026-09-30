# ADR 004 — Order recovery and idempotency

Date: 2026-09-30. Status: accepted Milestone 01 design baseline; implementation pending.

## Context

Order and Inventory cannot share a local database transaction. An HTTP timeout can occur after Inventory commits. Browsers can retry, and services can crash between durable steps. The first flow must preserve stock correctness without introducing a message broker.

## Decision

Order first commits a `PENDING` order, immutable lines, total, and customer-scoped submission key. Inventory then commits a unique terminal reservation decision for that order ID and all requested stock effects. Order applies the decision as `CONFIRMED` or `REJECTED`.

Inventory locks requested stock rows in consistent product-ID order, allocates every line or none, and stores rejections as durable decisions. Reservation uniqueness and payload comparison apply to successful and rejected decisions alike.

Order looks up an existing key before reading current catalog prices. Repeated identical requests return the saved order; changed content conflicts. Keep keys and decisions throughout this milestone.

A periodic reconciler inspects/repeats Inventory's operation for due pending orders. Conditional terminal updates make foreground and recovery processing safe to overlap. Never hold a database transaction during HTTP calls or infer a business rejection from a timeout.

Cancellation, expiry, release, and fulfillment are absent in 01. Confirmed orders retain allocated stock. Their later introduction requires coordinated states, migrations, and race tests.

## Alternatives

- Reserve before persisting Order risks allocations without a durable owning order.
- Releasing on every timeout could release a successful allocation while Order later confirms it.
- A distributed transaction or broker-driven saga would add technology beyond the first learning goal.
- In-memory retry state would disappear during restart and fail duplicate-submit recovery.

## Consequences

Temporary pending orders are visible to customers. Durable reconciliation work is required even in the first milestone. Retry safety relies on unchanged order identity and payload; it does not imply every response is identical or dependencies always recover.

## Verification

Real PostgreSQL last-item and multi-item tests; 100 same-key retries; changed-payload conflicts; crash after pending commit; lost reservation response; failed final Order update followed by successful recovery. [Order protocol](../milestone-01.md#6-order-and-reservation-protocol)
