---
name: spring-boot-backend
description: Design, implement, or debug VendorMargin Spring Boot APIs, services, persistence, migrations, and backend tests. Use for Java backend changes supporting POS import analytics or explicitly requested native POS features, not frontend-only tasks.
---

# Spring Boot Backend

## Scope and triggers

Use for backend endpoints, import jobs, analytics services, persistence changes, transaction defects, and Spring Boot integration tests.

VendorMargin's roadmap has two phases:

- Phase 1 imports and analyzes external POS exports.
- Phase 2 builds our own POS.

Default to Phase 1 unless the request calls for Phase 2. Preserve reusable sales and calculation boundaries, but do not prebuild checkout, tender processing, stock management, or offline synchronization. A skill invocation does not itself authorize deploying or changing live data.

## Workflow

1. Read repository guidance and inspect the build, Java/Spring Boot versions, package structure, API conventions, schema, and test setup. Use the existing Maven or Gradle wrapper. If no application exists, propose the minimal structure needed by the request; do not invent existing conventions or scaffold code for a planning-only task.
2. Define the requested API/service contract: inputs, validation, outputs, error behavior, persistence effects, and retry semantics. Keep HTTP DTOs separate from persistence entities. Match existing synchronous or job-based import patterns; introduce asynchronous infrastructure only when requirements justify it.
3. Keep controllers responsible for transport, services for use cases and transaction boundaries, and repositories for persistence. Keep vendor parsing outside core calculation rules. Load ../pos-data-importer/SKILL.md for import semantics or ../margin-calculator/SKILL.md for profitability rules when relevant.
4. Validate at the boundary and protect persistence invariants with appropriate constraints. Use BigDecimal for decimal money and explicit currency handling; distinguish event timestamps from business-local dates. Do not let server-default timezone determine reporting periods.
5. Design transaction boundaries around consistent units of work. Enforce import/business uniqueness at the database layer as well as in application logic, within the relevant business scope. For concurrent updates or retries, choose locking or idempotency mechanisms based on actual operations; avoid holding database transactions open across slow external calls.
6. Add versioned migrations using the project's established migration tool. Review compatibility with existing data and upgrade paths. Do not rely on automatic schema recreation outside disposable tests. Keep credentials in existing configuration mechanisms and out of source and logs.
7. Follow the repository's authentication and authorization model. When business/store scoping exists, enforce it on reads, writes, and import lookup keys. If access requirements are unspecified for a new exposed API, identify that decision before treating the endpoint as ready for use.
8. Make import failures diagnosable with batch identifiers, counts, and bounded error messages. Preserve useful exception causes internally without exposing raw POS data or stack traces in API responses. For reporting queries, inspect query count, pagination, and representative data volume where relevant.
9. For explicitly requested Phase 2 work, reuse validated domain contracts where semantics match. Define sale lifecycle, inventory/payment effects, concurrency, and retry boundaries only for the feature requested. Do not assume a database rollback can undo an external payment; specify compensation or reconciliation when external side effects are involved.

## Testing requirements

For implementation changes, select tests by the changed behavior:

- Unit tests for business rules and service failure paths; use independently determined expected amounts for monetary logic.
- HTTP/controller tests for validation, response shape, status codes, and the existing authorization rules.
- Repository integration tests against the production database engine when dialect, constraints, locking, or migrations matter; use the project's established disposable database setup.
- Migration tests for fresh schema and relevant upgrades with representative existing data.
- Import transaction tests for duplicates, concurrent submissions, partial failure, rollback, and retry behavior where affected.
- Reporting tests for aggregate correctness, business isolation where applicable, date boundaries, and pagination.
- For Phase 2 side effects, test duplicate commands and failure recovery using test doubles or sandbox integrations, never real payment execution.

Run the affected module's focused tests and relevant build checks using the repository wrapper. Report exact commands, results, and any infrastructure-related gaps; do not claim a passing build when tests were skipped or could not run. For design or skill-authoring requests, review the proposed contracts and test requirements without writing application code.