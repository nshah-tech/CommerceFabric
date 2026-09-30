# Milestone 01 API Contracts

Status: detailed planning specification for owner review. No endpoints have been implemented.

Related: [Milestone plan](../milestone-01.md), [Domain model](../domain-model.md), [API and identity decision](../adr/006-api-and-local-identity.md).

## 1. Contract conventions

- Public API base: `/api/v1` on Gateway. Business services expose the same resource paths to authenticated local callers. Gateway forwards only the public route allowlist.
- Internal API base: `/internal/v1` on the owning service; never forwarded by Gateway. Internal routes require a distinct per-caller service credential.
- JSON property names are camelCase. Successful resource responses are direct objects; list responses contain `items` and `nextCursor`. Errors have the shared shape below.
- JSON request bodies require `Content-Type: application/json`. Reject malformed JSON, unknown fields, incorrect types, unexpected bodies, and unsupported query parameters. Maximum JSON body size: 32 KiB.
- UUID inputs must have a valid hyphenated UUID representation; normalize case before comparison/hashing. Responses use lowercase UUIDs. Examples below use synthetic UUIDs.
- Money values are canonical decimal strings of nonnegative integer minor units, matching `0|[1-9][0-9]*`. No decimals, signs, spaces, or leading zeroes. Currency is `USD`.
- Quantities are JSON integers, never strings; no implicit coercion. Unit-price bounds: 0–100,000,000 cents. Stock-count bounds: 0–1,000,000,000. Order/reservation quantity bounds: 1–100.
- Timestamps are UTC ISO 8601. Nullable fields are explicitly `null` in responses rather than sometimes omitted.
- All responses carry `X-Request-ID`. Accept a caller UUID request ID or generate one; replace invalid values. Propagate it internally. Trace context uses standard tracing headers and is separate from customer identity.
- Public resources and errors use `Cache-Control: no-store`. No automatic retry of public writes with a new idempotency key.
- Health endpoints are local operational endpoints, separate from business routes. Public `/internal/*` requests return `404`.

### Authentication

Public business calls require `Authorization: Bearer <development-user-token>`. Token validation fixes the signing algorithm to RS256 and checks issuer `commercefabric-dev`, audience `commercefabric-api`, expiry, UUID subject, and role `customer` or `admin`. Proposed token lifetime is 30 minutes, with at most 30 seconds clock tolerance.

Gateway and each business service verify the user token. Gateway forwards the token and strips spoofable customer/role/service-auth headers. User tokens do not authorize internal operations.

Internal calls use `X-Service-Key`, selected from local configuration. Product accepts Order and Inventory caller keys only for lookup. Inventory accepts Order's caller key only for reservation operations. A user bearer token is insufficient for these operations; a service key is insufficient for public customer operations. Never log either credential.

The browser's identity fixtures are generated locally and accessible only through the development UI mechanism; no public token minting or login route is planned. CORS allows the configured local frontend origin, required authorization/idempotency headers, and exposes request-ID, location, and retry headers. No cookie authentication is used in this milestone.

### Shared error shape

```json
{
  "code": "VALIDATION_FAILED",
  "message": "The request contains invalid fields.",
  "requestId": "00000000-0000-4000-8000-000000000001",
  "details": [
    { "field": "items[0].quantity", "code": "OUT_OF_RANGE", "message": "Quantity must be between 1 and 100." }
  ]
}
```

`code`, `message`, and `requestId` are required. `details` is optional and contains safe validation information. A rejected accepted order additionally includes `order`, as specified below. Never include SQL, stack traces, credentials, or another customer's resource data.

| HTTP status | Stable codes | Meaning |
| --- | --- | --- |
| 400 | `VALIDATION_FAILED`, `INVALID_CURSOR`, `PRODUCT_NOT_AVAILABLE` | Request invalid or product cannot be accepted |
| 401 | `UNAUTHENTICATED` | User/service credential missing, invalid, or expired |
| 403 | `FORBIDDEN` | Valid identity lacks permission for this operation |
| 404 | `PRODUCT_NOT_FOUND`, `STOCK_NOT_FOUND`, `ORDER_NOT_FOUND`, `RESERVATION_NOT_FOUND`, `ROUTE_NOT_FOUND` | Resource unavailable in the caller's permitted scope |
| 409 | `SKU_EXISTS`, `STOCK_ALREADY_REGISTERED`, `STOCK_BELOW_RESERVED`, `PRODUCT_ARCHIVED`, `IDEMPOTENCY_CONFLICT`, `RESERVATION_CONFLICT`, `ORDER_REJECTED` | Conflict or a persisted business rejection |
| 413 | `REQUEST_TOO_LARGE` | Body exceeds the contract limit |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Body-bearing request is not JSON |
| 503 | `DEPENDENCY_UNAVAILABLE` | Operation cannot currently complete; accepted order behavior has a separate rule |
| 500 | `INTERNAL_ERROR` | Unexpected failure with a safe message and request ID |

Infrastructure errors must not masquerade as insufficient stock. A committed accepted order with an uncertain outcome should return its pending representation whenever the response path is still available.

## 2. Resources

### Product

```json
{
  "id": "00000000-0000-4000-8000-000000000101",
  "sku": "MUG-001",
  "name": "Learning Mug",
  "description": null,
  "priceMinor": "2500",
  "currency": "USD",
  "status": "ACTIVE",
  "createdAt": "2026-09-30T06:00:00.000Z",
  "updatedAt": "2026-09-30T06:00:00.000Z"
}
```

### Stock views

Customer stock view contains `productId`, `available`, and `observedAt`. Admin stock responses additionally contain `onHand` and `reserved`. `observedAt` is the read observation time, not a promise that stock remains available.

```json
{
  "productId": "00000000-0000-4000-8000-000000000101",
  "onHand": 10,
  "reserved": 2,
  "available": 8,
  "observedAt": "2026-09-30T06:01:00.000Z"
}
```

### Order

```json
{
  "id": "00000000-0000-4000-8000-000000000201",
  "status": "CONFIRMED",
  "currency": "USD",
  "totalMinor": "5000",
  "rejectionCode": null,
  "items": [
    {
      "productId": "00000000-0000-4000-8000-000000000101",
      "skuSnapshot": "MUG-001",
      "nameSnapshot": "Learning Mug",
      "unitPriceMinor": "2500",
      "quantity": 2,
      "lineTotalMinor": "5000"
    }
  ],
  "createdAt": "2026-09-30T06:01:00.000Z",
  "updatedAt": "2026-09-30T06:01:01.000Z"
}
```

Order does not expose submission hashes, credentials, recovery internals, or customer UUIDs. Ownership is already established by authorization. Item order is ascending normalized product ID. `rejectionCode` is null for pending/confirmed and `INSUFFICIENT_STOCK` or `STOCK_NOT_REGISTERED` for rejected.

## 3. Catalog routes

| Route | Caller | Request | Success | Endpoint-specific errors |
| --- | --- | --- | --- | --- |
| `GET /products` | Customer/admin | Optional `limit`, `cursor` | 200 active-product list | 400 invalid pagination |
| `GET /products/:id` | Customer/admin | UUID path | 200 active Product | 404 missing or archived |
| `POST /products` | Admin | Fields below | 201 Product; `Location: /api/v1/products/:id` | 409 SKU exists |
| `PATCH /products/:id` | Admin | One or more mutable fields below | 200 updated Product | 404 missing; 409 archived |
| `DELETE /products/:id` | Admin | UUID path; no body | 204, including repeat archive | 404 missing |

Create request requires `sku`, `name`, `priceMinor`, and `currency`; `description` is optional and may be null. Server chooses ID, status, and timestamps. SKU is trimmed and uppercased, then must match `[A-Z0-9][A-Z0-9._-]{0,63}`. Name is trimmed, nonblank, and 1–200 characters. Description is at most 2,000 characters. Currency must be exactly `USD`.

```json
{
  "sku": "MUG-001",
  "name": "Learning Mug",
  "description": null,
  "priceMinor": "2500",
  "currency": "USD"
}
```

PATCH allows only `name`, `description`, and `priceMinor`. Omitted fields stay unchanged; `description: null` clears description. Null name/price, empty patch, SKU/currency/status changes, and unknown fields are invalid. SKU conflicts do not revive archived products or recycle their identity.

Archive blocks future active lookup; it does not update orders or stock. Milestone 01 has no archived-product listing or reactivation endpoint.

## 4. Stock routes

| Route | Caller | Request | Success | Endpoint-specific errors |
| --- | --- | --- | --- | --- |
| `GET /inventory/:productId` | Customer/admin | UUID path | 200 role-appropriate stock view | 404 stock not registered |
| `POST /inventory/items` | Admin | `{ "productId": "UUID", "onHand": 10 }` | 201 admin stock view; resource location | 400 product unavailable; 409 already registered; 503 validation dependency unavailable |
| `PUT /inventory/:productId/stock` | Admin | `{ "onHand": 10 }` | 200 admin stock view | 404 missing; 409 below reserved |

Registration validates the product through Product's internal active lookup, starts with reserved zero, and rejects duplicate registration. PUT sets an absolute count under Inventory's lock; it never sets the reserved count. Repeating PUT with the same target is an absolute assignment, not an additive stock operation. Concurrent administrators use last committed assignment; there is no optimistic edit-version contract in this milestone.

Existing stock remains readable after product archival. Customers discover products through the active catalog, and new orders must pass Product's active lookup. A stock read does not reserve units.

## 5. Order routes and retry rules

| Route | Caller | Request | Success / business outcome |
| --- | --- | --- | --- |
| `POST /orders` | Customer | `Idempotency-Key` plus items | 201 first confirmed; 200 repeated confirmed; 202 pending; 409 rejected/conflicting |
| `GET /orders` | Customer | Optional `limit`, `cursor` | 200 list of own orders, with all states eligible |
| `GET /orders/:id` | Customer | UUID path | 200 own Order, regardless of pending/confirmed/rejected state |

POST requires exactly `items`, containing 1–20 distinct product IDs and quantities. Reject duplicate lines instead of silently merging. The browser must not send customer ID, price, total, order state, or order ID.

```json
{
  "items": [
    { "productId": "00000000-0000-4000-8000-000000000101", "quantity": 2 }
  ]
}
```

The key is 1–128 printable ASCII characters, with no leading/trailing whitespace or duplicate header values. The client generates it once per new submission and retains the key and immutable submitted payload across uncertainty/reloads. A deliberate second purchase uses a new key.

Order canonicalizes UUIDs, sorts lines, and hashes the normalized products/quantities. Existing customer/key lookup occurs before fetching prices. Same key and same request returns the same order; different content returns `409 IDEMPOTENCY_CONFLICT`, without changing the existing order.

| Situation | HTTP / body | Client behavior |
| --- | --- | --- |
| New order confirmed | 201 Order | Show confirmation |
| Existing confirmed order | 200 Order | Show existing confirmation |
| Accepted order still pending | 202 Order with status `PENDING` | Poll its location; preserve original key |
| Accepted order rejected | 409 error with `code = ORDER_REJECTED` and `order` containing the complete rejected Order | Show definitive rejection; a future deliberate submission gets a new key |
| Same key, different payload | 409 `IDEMPOTENCY_CONFLICT`; no existing order contents returned | Correct key/payload usage |
| Invalid/archived/unknown product before acceptance | 400 `PRODUCT_NOT_AVAILABLE`; no order | Fix selection; retain the original key if retrying the unchanged intent |
| Product/Order unavailable before acceptance | 503 `DEPENDENCY_UNAVAILABLE` | Outcome may be uncertain at Gateway; retry original key |
| Lost connection or Gateway timeout | 503 if a response is possible, otherwise no HTTP result | Never infer rejection; retry original key |

Every POST response containing an accepted order includes `Location: /api/v1/orders/:id`. A 202 also includes `Retry-After: 5`. The rejected error's `order.rejectionCode` distinguishes insufficient stock from unregistered stock; no such order field is included in a key conflict.

A 503 received from Gateway can occur after Order has accepted the request but before Gateway learns its identifier. The same-key retry resolves that ambiguity. Do not promise that every 503 means nothing committed.

GET on an unknown order or another customer's order returns the identical `404 ORDER_NOT_FOUND` shape. An admin role cannot use customer order routes merely by supplying a customer ID.

## 6. Pagination

Product and order lists use `limit` (decimal query integer 1–100; default 20) and optional opaque `cursor`. Sort descending by `(created_at, id)`; fetch subsequent rows strictly below the cursor pair. Return `nextCursor: null` when no later page is available.

Cursor payloads are versioned and integrity-protected, bound to the resource and, for orders, the verified customer. Reject tampered, malformed, foreign-customer, wrong-resource, or unsupported-version cursors with `400 INVALID_CURSOR`. Clients must not construct cursors. Limit may change between requests within bounds.

```json
{ "items": [], "nextCursor": null }
```

Pagination is a live view, not a frozen catalog/order snapshot. New records and archival can change what later pages contain. Immutable creation timestamps and stable tie-break IDs avoid reordering existing rows when prices or order statuses change.

## 7. Internal contracts

### Product lookup

`POST /internal/v1/products/lookup`, callers Order and Inventory.

Request: `{ "productIds": ["UUID"] }`, 1–20 distinct UUIDs. Inventory uses one ID for registration. Response 200: `{ "items": [Product] }`, ordered by normalized product ID. Every product must exist and be active; otherwise return `400 PRODUCT_NOT_AVAILABLE` for the complete lookup. No partial successful result. Authentication is still required even on localhost.

### Reserve inventory

`POST /internal/v1/reservations`, caller Order only.

Request requires `orderId` and `items`, with the same quantity/distinct-product bounds as Order. Order ID is generated and persisted by Order before calling Inventory.

```json
{
  "orderId": "00000000-0000-4000-8000-000000000201",
  "items": [
    { "productId": "00000000-0000-4000-8000-000000000101", "quantity": 2 }
  ]
}
```

First persisted decision returns 201, including a rejected decision. An identical repeat returns 200 with the saved decision. These transport codes indicate whether a decision resource was created; Order must inspect its `status` for the business outcome.

```json
{
  "id": "00000000-0000-4000-8000-000000000301",
  "orderId": "00000000-0000-4000-8000-000000000201",
  "status": "RESERVED",
  "rejectionCode": null,
  "items": [
    { "productId": "00000000-0000-4000-8000-000000000101", "quantity": 2 }
  ],
  "createdAt": "2026-09-30T06:01:00.000Z"
}
```

For a rejection, set `status = REJECTED` and a non-null code. Deterministic rejection precedence: missing stock row → `STOCK_NOT_REGISTERED`; otherwise insufficient available quantity → `INSUFFICIENT_STOCK`. Return all requested lines, without stock effects.

Same order ID with changed quantities/products returns `409 RESERVATION_CONFLICT`. Transaction/lock failures return 503 if the service can respond; they do not create a rejected business decision. An unknown HTTP result is recovered by inspect/retry with the same order ID.

### Inspect reservation

`GET /internal/v1/reservations/by-order/:orderId`, caller Order only. Return 200 with the decision shape above, or `404 RESERVATION_NOT_FOUND` if no committed decision is visible.

A 404 may race with an in-flight reservation transaction; Order may repeat the idempotent reserve operation. It must not create a new order ID or mark the order rejected merely because the decision was temporarily absent.

## 8. Operational behavior

- Services bind to loopback locally: Gateway 3000, Product 3001, Inventory 3002, Order 3003, frontend 5173, PostgreSQL 5432. Ports are configurable and no other environment is modified by planning.
- `/health/live`: 200 `{ "status": "ok" }` when the process can respond; no dependency calls.
- `/health/ready`: 200 `{ "status": "ready" }` or 503 `{ "status": "not_ready" }`; business services check their own database and required applied migration history, not downstream business services. Gateway checks required configuration. No schema mutations or detailed credentials/errors in probes.
- Order-to-Product/Inventory and Inventory-to-Product calls use the milestone's initial 2-second timeout. Gateway-to-service timeout starts at 8 seconds, allowing the two sequential order dependencies to return a pending result. Values remain tunable experiment parameters.
- No unbounded immediate HTTP retries. Inventory transient transaction retries stay within the request deadline. Accepted-order reconciliation runs on durable pending rows as described in the milestone plan.
- Log safe IDs, operation, status, duration, and error codes. Exclude tokens, service keys, full bodies, submission keys, and database URLs. Hashes and customer identifiers are not public telemetry labels.

## 9. Contract verification and evolution

During implementation, maintain OpenAPI 3.0.3 contracts under `contracts/openapi/` for Gateway, Product, Inventory, and Order. This Markdown specification provides planning examples and rules; it is not a claim that generated or executable schemas exist yet.

Contract checks must cover success and negative cases, nested unknown fields, all numeric bounds, archived-product semantics, deterministic list ordering, cursor scoping, public/internal separation, role/customer isolation, and persisted decisions across timeout/retry boundaries.

OpenAPI validation alone does not prove transaction correctness. Real PostgreSQL tests verify stock invariants and concurrency. Browser tests verify 202 polling and error handling instead of treating every non-2xx as an unaccepted request.

No breaking changes are silently introduced to `/v1`. A new milestone documents compatible extension or explicit versioning and the old/new overlap period. No future cancellation, expiry, or payment endpoints are predesigned here.
