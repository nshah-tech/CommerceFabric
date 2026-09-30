# Milestone 01 Technology and Compatibility Matrix

Status: planning selection. Latest-stable policy and TypeScript exception approved by the owner on 2026-09-30. Published package metadata checked on that date; no packages installed or application/runtime tests run.

Related: [Milestone plan](../milestone-01.md), [tooling ADR](../adr/007-initial-tooling.md), [metadata evidence](milestone-01-version-evidence.json).

## 1. Version policy

Use the latest stable release when introducing a technology. Exclude prereleases and beta database releases. Record exact versions, their sources, and compatible package families. During authorized scaffolding, pin direct dependencies and commit the resolved lockfile; do not use floating `latest` container tags or automatically change the toolchain on every run.

Future Go, Python, broker, Kubernetes, and cloud versions are selected when their milestone starts. Recheck this dated selection before installation if work starts later. Report any newly discovered incompatibility before changing an accepted version policy.

## 2. Runtime and host tools

| Component | Planning selection | Evidence / purpose |
| --- | --- | --- |
| Node.js | **26.10.0** | [Official current-release checksums](https://nodejs.org/download/release/latest/SHASUMS256.txt); latest stable current line, rather than the earlier Node 24 LTS proposal |
| npm | **12.1.0** | [Published metadata](https://registry.npmjs.org/npm/12.1.0); workspace package manager |
| PostgreSQL | **18.6** | [Official supported-version table](https://www.postgresql.org/support/versioning/); PostgreSQL 19 beta excluded |
| Docker Engine / CLI | **29.8.1** | [Official release notes](https://docs.docker.com/engine/release-notes/29/); local database and isolated test containers |
| Docker Compose plugin | **5.5.1** | [Official stable release](https://github.com/docker/compose/releases/tag/v5.5.1); use `docker compose` |
| Git | **2.56.0** | [Official release distribution](https://www.kernel.org/pub/software/scm/git/); record the actual installed host version in Stage A evidence |
| GNU Make | **4.4.1** | [Official stable distribution listing](https://ftp.gnu.org/gnu/make/); task interface |

Runtime/container versions are selected, not installed. Resolve PostgreSQL's platform-specific image digest during Stage A and use the same image for local and integration tests. Docker version compatibility with the host OS and the Testcontainers client needs an actual startup check. Do not silently substitute an older host version; report platform/package-provider conflicts.

## 3. Exact JavaScript package selection

Links below point to version-specific publisher metadata. Packages can be runtime or development dependencies and belong only in the workspaces that use them. Optional peers for unused database drivers, adapters, bundlers, and browser features do not need installation.

| Package | Selected version |
| --- | --- |
| [`@nestjs/common`](https://registry.npmjs.org/@nestjs/common/12.1.1) | 12.1.1 |
| [`@nestjs/core`](https://registry.npmjs.org/@nestjs/core/12.1.1) | 12.1.1 |
| [`@nestjs/platform-express`](https://registry.npmjs.org/@nestjs/platform-express/12.1.1) | 12.1.1 |
| [`@nestjs/testing`](https://registry.npmjs.org/@nestjs/testing/12.1.1) | 12.1.1 |
| [`@nestjs/cli`](https://registry.npmjs.org/@nestjs/cli/12.0.8) | 12.0.8 |
| [`@nestjs/schematics`](https://registry.npmjs.org/@nestjs/schematics/12.0.6) | 12.0.6 |
| [`@nestjs/typeorm`](https://registry.npmjs.org/@nestjs/typeorm/12.0.2) | 12.0.2 |
| [`@nestjs/swagger`](https://registry.npmjs.org/@nestjs/swagger/12.0.2) | 12.0.2 |
| [`typeorm`](https://registry.npmjs.org/typeorm/1.1.1) | 1.1.1 |
| [`pg`](https://registry.npmjs.org/pg/8.23.0) | 8.23.0 |
| [`typescript`](https://registry.npmjs.org/typescript/6.0.3) | 6.0.3 |
| [`react`](https://registry.npmjs.org/react/19.3.0) | 19.3.0 |
| [`react-dom`](https://registry.npmjs.org/react-dom/19.3.0) | 19.3.0 |
| [`vite`](https://registry.npmjs.org/vite/8.3.1) | 8.3.1 |
| [`@vitejs/plugin-react`](https://registry.npmjs.org/@vitejs/plugin-react/6.1.1) | 6.1.1 |
| [`jest`](https://registry.npmjs.org/jest/30.5.2) | 30.5.2 |
| [`@types/jest`](https://registry.npmjs.org/@types/jest/30.0.0) | 30.0.0 |
| [`@swc/core`](https://registry.npmjs.org/@swc/core/1.16.12) | 1.16.12 |
| [`@swc/jest`](https://registry.npmjs.org/@swc/jest/0.2.39) | 0.2.39 |
| [`supertest`](https://registry.npmjs.org/supertest/7.3.0) | 7.3.0 |
| [`@types/supertest`](https://registry.npmjs.org/@types/supertest/7.2.1) | 7.2.1 |
| [`testcontainers`](https://registry.npmjs.org/testcontainers/12.2.0) | 12.2.0 |
| [`@testcontainers/postgresql`](https://registry.npmjs.org/@testcontainers/postgresql/12.2.0) | 12.2.0 |
| [`@playwright/test`](https://registry.npmjs.org/@playwright/test/1.63.0) | 1.63.0 |
| [`eslint`](https://registry.npmjs.org/eslint/10.11.0) | 10.11.0 |
| [`typescript-eslint`](https://registry.npmjs.org/typescript-eslint/8.71.0) | 8.71.0 |
| [`prettier`](https://registry.npmjs.org/prettier/3.9.9) | 3.9.9 |
| [`class-validator`](https://registry.npmjs.org/class-validator/0.15.1) | 0.15.1 |
| [`class-transformer`](https://registry.npmjs.org/class-transformer/0.5.1) | 0.5.1 |
| [`reflect-metadata`](https://registry.npmjs.org/reflect-metadata/0.2.2) | 0.2.2 |
| [`rxjs`](https://registry.npmjs.org/rxjs/7.8.2) | 7.8.2 |
| [`pino`](https://registry.npmjs.org/pino/10.3.1) | 10.3.1 |
| [`jsonwebtoken`](https://registry.npmjs.org/jsonwebtoken/9.0.3) | 9.0.3 |
| [`@types/jsonwebtoken`](https://registry.npmjs.org/@types/jsonwebtoken/9.0.10) | 9.0.10 |
| [`@opentelemetry/api`](https://registry.npmjs.org/@opentelemetry/api/1.9.1) | 1.9.1 |
| [`@opentelemetry/sdk-node`](https://registry.npmjs.org/@opentelemetry/sdk-node/0.222.0) | 0.222.0 |
| [`@opentelemetry/auto-instrumentations-node`](https://registry.npmjs.org/@opentelemetry/auto-instrumentations-node/0.80.0) | 0.80.0 |
| [`@opentelemetry/exporter-trace-otlp-http`](https://registry.npmjs.org/@opentelemetry/exporter-trace-otlp-http/0.222.0) | 0.222.0 |
| [`@types/node`](https://registry.npmjs.org/@types/node/26.6.3) | 26.6.3 |
| [`@types/react`](https://registry.npmjs.org/@types/react/19.3.0) | 19.3.0 |
| [`@types/react-dom`](https://registry.npmjs.org/@types/react-dom/19.3.0) | 19.3.0 |
| [`@apidevtools/swagger-parser`](https://registry.npmjs.org/@apidevtools/swagger-parser/13.1.0) | 13.1.0 |
| [`@types/pg`](https://registry.npmjs.org/@types/pg/8.23.1) | 8.23.1 |
| [`@eslint/js`](https://registry.npmjs.org/@eslint/js/10.0.1) | 10.0.1 |
| [`openapi-types`](https://registry.npmjs.org/openapi-types/12.1.3) | 12.1.3 |
| [`@opentelemetry/core`](https://registry.npmjs.org/@opentelemetry/core/2.11.0) | 2.11.0 |
| [`npm`](https://registry.npmjs.org/npm/12.1.0) | 12.1.0 |
| [`@opentelemetry/sdk-trace-base`](https://registry.npmjs.org/@opentelemetry/sdk-trace-base/2.11.0) | 2.11.0 |

This list deliberately includes Node 26 type definitions, matched React/ReactDOM/type definitions, required OpenTelemetry peers, and OpenAPI parser types. npm's package entry records the package-manager version; it is not a business-service dependency.

## 4. Approved TypeScript exception

Latest stable TypeScript is **7.0.2**, but:

| Consumer | Published TypeScript peer range | Result with 7.0.2 |
| --- | --- | --- |
| Nest Swagger 12.0.2 | `^5.5.0 || ^6.0.0` | Outside supported range |
| typescript-eslint 8.71.0 | `>=4.8.4 <6.1.0` | Outside supported range |

The owner approved **TypeScript 6.0.3** as the one compatibility exception. It satisfies both ranges, as well as Nest schematics' minimum. Nest CLI also declares a TypeScript `~6.0.2` dependency. Use the selected TypeScript across the initial backend and frontend typechecks.

When the consumers support TypeScript 7, assess the compiler/tooling changes and remove the exception through a reviewed upgrade. Do not force installation past peer conflicts. [Nest Swagger metadata](https://registry.npmjs.org/@nestjs/swagger/12.0.2), [typescript-eslint metadata](https://registry.npmjs.org/typescript-eslint/8.71.0)

## 5. Compatibility result and limits

Semantic-version checks over the recorded **48 packages** found no selected Node-engine, npm-engine, required-peer-presence, or selected peer-version conflicts: **29 Node engine ranges** and **43 peer version ranges** were checked.

Specific alignments:

- Nest common/core/platform/testing packages share 12.1.1; CLI/schematics/adapter/Swagger have their separately published compatible versions.
- Nest TypeORM adapter 12.0.2 declares `^0.3.0 || ^1.0.0-dev`; stable TypeORM 1.1.1 satisfies that range. Its Node and PostgreSQL-driver requirements also match the selection.
- React and ReactDOM are matched; Vite and its React plugin peer ranges match.
- Jest 30.5.2 uses SWC for transformation, with TypeScript typechecking separately. No ts-jest version is selected.
- The OpenTelemetry API/core/SDK peers are explicitly present. Console span export supplies the initial trace demonstration; OTLP export is optional configuration.

These checks cover direct published metadata, not a complete installed transitive graph, security audit, native-binary/OS compatibility, or runtime correctness. No claim that the whole stack already runs is made.

## 6. Module format and TypeORM 1 conventions

Backend workspace packages use CommonJS, TypeScript `NodeNext` module/resolution settings with explicit package format, ES2023 target, and decorator metadata. Selected Node satisfies Nest's documented requirements for loading ESM-only framework packages from CommonJS and using Jest. Frontend is a separate ESM/Vite workspace. [Nest migration guide](https://docs.nestjs.com/migration-guide)

Configure SWC tests for legacy decorators and emitted decorator metadata, matching the production compiler. Run TypeScript typechecking independently; SWC transformation is not typechecking. [SWC Jest guidance](https://swc.rs/docs/usage/jest)

Use TypeORM 1's explicit `DataSource`, current repository/query APIs, and explicit environment configuration. Avoid removed global Connection APIs and legacy environment auto-loading. Treat invalid null/undefined filters as errors, specify relation behavior, and review SQL rather than relying on old tutorials. [TypeORM 1 release guidance](https://typeorm.io/blog/typeorm-1-0/)

Disable automatic schema synchronization, automatic startup migrations, and unneeded automatic extension installation. Compiled entities and migrations must be loaded exactly once, without stale source/dist duplication.

## 7. Checks after implementation authorization

Stage A must record:

1. Actual runtime/package-manager/host versions and platform; refresh stale release metadata.
2. A clean dependency resolution without forced peer bypass; committed lockfile.
3. Independent backend builds and typechecks with the selected module format.
4. Nest dependency injection and validation/decorator metadata under production and Jest/SWC paths.
5. TypeORM 1 DataSource/entity loading, migration execution/locking, and PostgreSQL 18 container integration.
6. Vite frontend build and Playwright's matching browser installation.
7. Testcontainers/Docker startup and cleanup on the actual host.
8. Focused dependency/security findings and any new incompatibility before proceeding.

These are implementation checks, not unfinished application work hidden in the planning status. Planning selects an evidence-backed stack; installation and execution remain separately authorized.
