# ADR 006 — REST contracts and local identities

Date: 2026-09-30. Status: detailed refinement of the accepted design, ready for owner review; implementation pending.

## Context

Customer isolation and service authentication must be testable before AI or external identity providers are introduced. Gateway timeouts and stock decisions need unambiguous client contracts.

## Decision

Use versioned REST/JSON contracts, documented in the [API specification](../api-contracts/milestone-01.md), with OpenAPI 3.0.3 artifacts produced during implementation. Validate unknown fields, money strings, bounded integer quantities, and UUIDs explicitly.

Public routes use short-lived RS256 development tokens with verified issuer, audience, subject, expiry, and role. Generate two synthetic customers and an admin locally. Services derive ownership from verified claims, not forwarded identity headers. Local fixtures and signing keys are excluded from Git and later production builds.

Internal routes use distinct per-caller service keys and endpoint allowlists. They are unreachable through public Gateway routing. Local HTTP is loopback-only; production identity/TLS/rotation requires a later plan.

Use a common safe error envelope. Return 202 for known pending orders and a persisted rejected order with 409. Internal reservation creation returns a decision resource whose status determines the business result. Retry an uncertain public order with its original key even when Gateway cannot return an order ID.

Use bounded keyset pagination with customer/resource-scoped integrity-protected cursors. Limit catalog/stock admin operations to admin and order access to the owning customer.

## Alternatives

- Trusting customer headers would allow callers to choose ownership.
- Building a full identity service would expand milestone scope without improving the initial stock/recovery exercise.
- Sharing one user/service credential would erase the public/internal operation boundary.
- Treating every non-2xx as an unaccepted order would cause duplicate submissions after Gateway uncertainty.

## Consequences

Development authentication is an explicit local fixture mechanism, not production authentication. Clients must distinguish transport errors, pending work, and durable rejection. Cursor handling is part of authorization and must not leak another customer's data.

## Verification

Algorithm/issuer/audience/expiry checks; role and customer isolation; forged-header rejection; internal caller allowlists; cursor tampering/scoping; 202 polling and same-key Gateway timeout recovery.
