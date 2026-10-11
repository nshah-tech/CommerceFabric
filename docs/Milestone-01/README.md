# Milestone 01 Documentation

All planning, decisions, and supporting evidence for Milestone 01 live in this folder. The shared [Roadmap](../ROADMAP.md) describes the overall sequence; the [Master Plan](../../CommerceFabric_Master_Plan.md) defines the project's goal.

Status: planning package complete and ready for final owner review. Implementation remains pending authorization. Version evidence retains its original check date; this folder reorganization does not refresh releases or execute runtime checks.

## Reading order

| Document | Purpose |
| --- | --- |
| [Milestone-01.md](Milestone-01.md) | Scope, service architecture, order workflow, implementation stages, and acceptance criteria |
| [Domain-Model-01.md](Domain-Model-01.md) | Business concepts, relationships, states, invariants, and worked examples |
| [Database-01.md](Database-01.md) | Current tables, types, constraints, relationships, and indexes |
| [API-Contracts-01.md](API-Contracts-01.md) | Requests, responses, validation, authorization, errors, and retry behavior |
| [Technology-01.md](Technology-01.md) | Selected tooling, compatibility rules, and approved TypeScript exception |
| [Database-Operations-01.md](Database-Operations-01.md) | Permissions, migrations, backup/restore, connection budgets, and recovery |
| [ADR/README.md](ADR/README.md) | Decision registry and trade-offs |
| [Planning-Review-01.md](Planning-Review-01.md) | Consistency review, evidence limits, and remaining gates |
| [Technology-01-Version-Evidence.json](Technology-01-Version-Evidence.json) | Dated package metadata and compatibility results |
| [Experiments/TEMPLATE.md](Experiments/TEMPLATE.md) | Template for future experiments and implementation evidence |

## Folder structure

```text
Milestone-01/
├── README.md
├── Milestone-01.md
├── Domain-Model-01.md
├── Database-01.md
├── Database-Operations-01.md
├── API-Contracts-01.md
├── Technology-01.md
├── Technology-01-Version-Evidence.json
├── Planning-Review-01.md
├── ADR/
│   ├── README.md
│   └── 001–007 decision records
└── Experiments/
    └── TEMPLATE.md
```

Future milestones use `Milestone-02/`, `Milestone-03/`, and so on, with their own numbered documents, ADRs, and experiments. Link to earlier milestone decisions when relevant; design database changes only for the milestone currently being planned.
