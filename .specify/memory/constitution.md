<!--
Sync Impact Report
- Version change: N/A -> 1.0.0
- Modified principles: N/A (initial adoption)
- Added sections: Core Principles, Quality Gates, Workflow Expectations, Governance
- Removed sections: N/A
- Templates requiring updates: ✅ .specify/templates/plan-template.md, ✅ .specify/templates/tasks-template.md
- Follow-up TODOs: TODO(COMMAND_TEMPLATES): .specify/templates/commands/ is missing; confirm if required.
-->

# spec-kit-lab Constitution

## Core Principles

### I. RESTful Design

- All endpoints MUST be resource-oriented, use standard HTTP verbs, and remain stateless.
- Status codes and error responses MUST follow REST conventions and be consistent.
- RPC-style endpoints are NOT allowed unless documented as exceptions in the spec.
  Rationale: A consistent REST surface reduces client complexity and enables caching.

### II. Documentation Clarity

- Every endpoint, request/response schema, and error model MUST be defined in OpenAPI 3.0.1.
- The specification MUST remain in sync with behavior; undocumented behavior is forbidden.
  Rationale: The spec is the contract for clients and tests.

### III. Testability

- Every feature MUST include unit tests that cover success and error paths.
- Critical game mechanics (dice rolls, modifiers, combat rules) MUST have focused unit tests.
- Tests MUST run in CI and pass before merge.
  Rationale: Reliable mechanics and stable APIs depend on repeatable tests.

### IV. Simplicity

- Prefer the simplest design that meets requirements; avoid speculative abstractions.
- Additional layers or patterns MUST be justified in the implementation plan.
  Rationale: Simplicity keeps the codebase maintainable for a learning lab.

### V. Performance

- API endpoints MUST target p95 latency under 200ms for nominal load.
- Performance budgets MUST be documented in plan.md when endpoints are added or changed.
  Rationale: The game must remain responsive during interactive play.

## Quality Gates

- OpenAPI 3.0.1 coverage is complete for every endpoint and schema.
- Unit tests are present for every feature and pass in CI.
- REST conventions are verified for new or changed endpoints.
- Performance targets are evaluated when a change affects response time.

## Workflow Expectations

- Work follows the Spec Kit workflow: `/speckit.specify` -> `/speckit.plan` -> `/speckit.tasks` -> `/speckit.implement`.
- The plan MUST include a Constitution Check section that references the five principles.
- Deviations from principles MUST be documented with rationale and mitigation.

## Governance

- This constitution supersedes other guidance documents.
- Amendments require a documented rationale, updated version, and reviewer approval.
- Versioning follows semantic versioning: MAJOR for breaking governance changes,
  MINOR for new principles or substantial expansions, PATCH for clarifications.
- Every plan and PR review MUST confirm compliance or document exceptions.

**Version**: 1.0.0 | **Ratified**: 2026-02-06 | **Last Amended**: 2026-02-06
