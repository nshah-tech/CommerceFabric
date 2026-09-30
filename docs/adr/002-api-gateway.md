# ADR 002 — Public API Gateway

Date: 2026-09-30. Status: accepted Milestone 01 design baseline; implementation pending.

## Context

The React application needs one public endpoint and consistent authentication, request correlation, and error behavior. The domain services still need to own their business rules and verify callers.

## Decision

Use a small NestJS Gateway exposing `/api/v1`. Route allowlisted product operations to Product, stock operations to Inventory, and order operations to Order. Preserve safe resource responses and documented status codes. Define finite downstream deadlines and safe errors for unavailable upstreams.

Gateway verifies user tokens, strips spoofable identity/internal-auth headers, forwards the original bearer token, and propagates request IDs and trace context. Services also verify user tokens and enforce their own role/customer authorization.

Do not expose `/internal/v1` through Gateway. Gateway has no business database, catalog cache, inventory counter, pricing calculation, or order state machine. Order directly calls Product and Inventory for its domain flow.

## Alternatives

- Direct browser-to-service calls would distribute URLs/authentication and expose internal topology to the UI.
- A general reverse proxy would reduce custom routing work but provide fewer opportunities to learn API policy and Nest integration initially.
- Orchestrating order logic in Gateway would detach the business operation from its durable Order state.

## Consequences

Gateway is an additional failure point. A Gateway timeout cannot prove that the downstream write did not commit; clients retry order submissions using the original key. Different business services can evolve independently behind explicit contracts.

## Verification

Internal routes are inaccessible through Gateway; forged customer headers do not change ownership; upstream timeout tests retain request correlation and safe retry semantics. [API contracts](../api-contracts/milestone-01.md)
