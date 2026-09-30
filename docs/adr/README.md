# Architecture Decision Records

These records explain Milestone 01's decisions and trade-offs. The owner accepted its overall design and latest-stable version policy on 2026-09-30, including the TypeScript compatibility exception. Detailed contract refinements remain available for final planning review. No record authorizes implementation.

| Record | Decision | Planning status |
| --- | --- | --- |
| [001](001-microservice-boundaries.md) | Separate Gateway, Product, Inventory, and Order processes | Accepted baseline |
| [002](002-api-gateway.md) | One public gateway with business logic owned by services | Accepted baseline |
| [003](003-service-database-ownership.md) | One PostgreSQL instance, three separately owned databases | Accepted baseline |
| [004](004-order-recovery-and-idempotency.md) | Durable pending order and idempotent Inventory decision | Accepted baseline |
| [005](005-database-migrations.md) | Explicit service-owned schema migrations and recovery exercises | Accepted baseline |
| [006](006-api-and-local-identity.md) | Versioned REST, development identities, and distinct internal authentication | Detailed refinement for review |
| [007](007-initial-tooling.md) | Latest stable tooling with one approved TypeScript exception | Policy accepted; metadata checked |

Use this index as the identifier registry when future milestones add records. Supersede an accepted decision with a new record and links; retain the original reasoning.
