# ADR 007 — Initial tooling and version policy

Date: 2026-09-30. Status: latest-stable policy and TypeScript exception accepted by the owner; version metadata checked; implementation pending.

## Context

This is a new learning project. The owner prefers latest stable technology/tooling and wants declared compatibility issues surfaced rather than quietly selecting older releases. Selecting each package's latest release independently can violate peer requirements.

## Decision

Use the [version matrix](../technology/milestone-01.md) as the dated selection. Node 26.10.0 and npm 12.1.0 replace the earlier Node 24 LTS proposal. Use Nest 12.1.1, TypeORM 1.1.1, PostgreSQL 18.6, React 19.3.0, and the listed current stable tooling.

Exception approved by the owner: TypeScript 6.0.3. Latest TypeScript 7.0.2 falls outside the current Nest Swagger and typescript-eslint peer ranges. Revisit the exception when both support it; do not bypass peer checks with force/legacy-peer installation flags.

Backend packages use CommonJS, TypeScript NodeNext resolution with explicit package format, ES2023 target, legacy decorator metadata, and Jest 30.5.2 with SWC transformation configured for decorators/metadata. Typecheck separately with TypeScript. Frontend uses Vite's ESM workflow with bundler resolution and React JSX.

Use npm workspaces without a monorepo orchestration framework. Build each service independently. Use a standalone compiled TypeORM migration DataSource and controlled runner, native Node environment loading, explicit query transactions, and direct Pino/OpenTelemetry integration through small technical adapters.

Initial traces can use OpenTelemetry's console span exporter, avoiding a full observability platform before Milestone 08. OTLP export is a configured option when a local backend is introduced; no new trace-server version is silently added.

Pin exact direct versions and commit a lockfile during authorized scaffolding. Select current stable versions when future milestone technologies are introduced. Recheck stale version selections before installation, report new conflicts, and preserve the approved exception until compatibility changes.

## Alternatives

- Node 24 LTS is a reasonable supported runtime, but the accepted project preference is the latest stable release, including the current release line.
- Independently using every latest package would produce a declared TypeScript peer conflict.
- Older TypeORM 0.3 has no established necessity for this new project; the current adapter supports TypeORM 1.
- ESM backend/Vitest is an alternative, but CommonJS/Jest retains the master plan's initial testing approach while modern framework packages are loaded under the selected runtime.

## Consequences

Published metadata compatibility is established; installation, builds, decorators, Jest interop, native binaries, and database behavior are not yet runtime-verified. These are implementation Stage A checks. The exception is explicit and does not justify using old versions elsewhere without evidence.

## Verification

Persist metadata and semantic-version comparison results in the version evidence. After implementation authorization, verify clean dependency resolution, independent builds/typechecks, Nest injection, migration DataSource loading, Jest/SWC behavior, Vite build, and PostgreSQL container startup before business implementation.
