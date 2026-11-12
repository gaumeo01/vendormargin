# VendorMargin Project Guidance

## Scope and source of truth

VendorMargin Phase 1 analyzes sales and purchase exports from existing POS/supplier systems with user-defined recipes. Phase 2 builds a proprietary POS using the same analytics engine.

Keep Phase 1 strictly export-based. Do not add order entry, checkout, payments, receipt issuance, cash drawers, operational stock tracking, offline POS synchronization or speculative POS modules. Reusable normalized sales and calculation contracts are sufficient preparation for Phase 2.

Read [docs/PRD.md](docs/PRD.md) for requirements, formulas, assumptions and open questions; [docs/DATABASE.md](docs/DATABASE.md) for proposed data relationships; [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for proposed boundaries; and [docs/ROADMAP.md](docs/ROADMAP.md) for priorities and capacity gates. These documents are a validated-format-first proposal, not evidence that implementation or all defaults are approved.

Confirmed stack: Java 21, Spring Boot, PostgreSQL, Flyway; React, TypeScript, Vite, Tailwind CSS; JUnit, Testcontainers, Playwright; Docker Compose and GitHub Actions. Library versions, build tool, auth provider and deployment services remain open until selected and recorded.

The current authorized work is documentation only. Do not infer permission to scaffold or implement features from this file. Follow subsequent explicit implementation requests when they arrive. Preserve task scope and user edits.

## Working conventions

- Inspect existing code/build conventions before changes. The initial repository contains skills and planning documents, not an application. Do not claim suggested paths or commands already exist.
- Plan for one part-time developer. Prefer one backend, one database, simple forms/tables and a small vertical slice. Validate one representative sales and purchase path before broad adapters; a second source is conditional.
- Preserve P0 correctness, tenant isolation and truthful coverage when cutting scope. Price alerts, extra formats/sources and richer dashboards can move beyond week 14 with explicit disclosure.
- Label unconfirmed choices as assumptions or open questions and update the PRD register with evidence when resolved. Do not invent POS field semantics, sample data facts or business approval.
- Keep business rules in services/calculation functions and vendor parsing at import boundaries. Avoid premature abstractions or new infrastructure without a demonstrated requirement.
- Add only files needed for the active task. Do not introduce migrations, application code, dependencies or CI configuration during documentation-only work.

## Relevant repository skills

- Use [.agents/skills/pos-data-importer/SKILL.md](.agents/skills/pos-data-importer/SKILL.md) for format mapping, import validation, provenance, reconciliation and deduplication. Its sales-oriented guidance must also respect the PRD's distinct purchase semantics.
- Use [.agents/skills/margin-calculator/SKILL.md](.agents/skills/margin-calculator/SKILL.md) for monetary calculations. VendorMargin reports estimated ingredient costs/margins, not verified accounting COGS or net profit; apply the PRD's confirmed business decisions as they are validated.
- Use [.agents/skills/spring-boot-backend/SKILL.md](.agents/skills/spring-boot-backend/SKILL.md) for backend/API/persistence work. Global frontend/testing skills may be used when applicable and available; do not assume optional tools are installed.

## Data and calculation rules

- Use BigDecimal and PostgreSQL numeric for money and quantities; expose exact API decimals without JavaScript floating-point recalculation. Keep currency/unit/timezone explicit.
- Unknown cost is distinct from zero. Show missing inputs, cost coverage and known-cost subset denominators. Never average item margin percentages.
- Preserve source identifiers as text, source row/batch lineage, original date semantics and the mapping version. Never double count summary/detail rows or apply tax/refund adjustments twice.
- Recipe and price selection must respect effective dates and the documented policy. Do not use future/current prices silently for past sales. Purchases are cost observations, not inventory movements.
- Test compatible unit conversions, pack changes and yield. Reject unsupported dimensional conversions or modifiers instead of guessing.
- Keep financial import commits atomic and retry-safe. Exact hashes alone do not establish overlap safety. Conflicting corrections require a validated policy.
- Published recipe contents and completed calculation runs retain historical provenance. New inputs make prior reports stale rather than silently rewriting viewed results.

## Tenant and persistence conventions

- Every business query/mutation requires validated tenant context. IDs, request tenant headers and obscurity are not authorization. Scope artifacts, job/status paths and audit access too.
- Follow the validated tenancy design, including composite tenant foreign keys and tested database isolation where adopted. Test with the restricted runtime role, not migration/admin privileges.
- Flyway owns schema changes when implementation starts. Test fresh and upgrade paths on PostgreSQL Testcontainers. Do not edit an applied migration or use destructive automatic schema creation against persistent data.
- Do not commit credentials or real POS exports/customer data. Use sanitized/synthetic fixtures. Bound file parsing resources; never execute macros or spreadsheet formulas.
- Preserve history when archiving catalogue records. Destructive retention/deletion and production changes need explicit task scope and a concrete recovery plan.

## Quality gates for implementation

Choose checks for the changed behavior; use actual repository wrappers/scripts once they exist. Do not fabricate successful commands or create tests that merely mirror implementation wording.

1. **Business logic:** independent JUnit examples for conversions, recipes, costs, discounts, tax, refunds, zero/negative denominators, missing costs and aggregate margins.
2. **Imports:** golden fixtures for supported formats; malformed/over-limit files; exact/overlapping reimports; correction conflicts; failure rollback and concurrent retry. Reconcile counts and totals independently.
3. **Database/security:** PostgreSQL Testcontainers migrations, tenant FK/RLS behavior where adopted, two-tenant API tests, native query and pooled-connection paths, unauthorized resource/artifact access.
4. **Frontend:** TypeScript checks/build plus Playwright critical import-to-dashboard workflow and relevant error/empty/partial/stale states. Verify basic keyboard use and narrow viewport behavior for changed UI.
5. **Release:** all P0 acceptance criteria, operator-validated source semantics and recipe examples, restore rehearsal, access revocation, upload limits and documented performance measurements. No unresolved tenant-isolation/data-integrity blocker.

Run focused tests during changes and required CI gates before release. Record commands, pass/fail/skipped results and untested limitations. A skipped test is not a pass. Update relevant docs when business contracts change.

## Documentation quality gate

For documentation-only changes, check links, terminology, priority/phase consistency, assumption labels, entity/formula agreement and measurable acceptance criteria. Do not run or claim application tests when no application exists. Keep exactly the requested documentation scope unless the user authorizes more.
