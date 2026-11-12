# VendorMargin 14-Week Implementation Backlog

Status: proposed capacity-based plan, not a delivery guarantee. This document authorizes no implementation by itself. Scope and acceptance criteria are in [PRD.md](PRD.md); architecture and data design remain proposals until validated.

## 1. Capacity and cut line

**Confirmed:** one solo developer working part-time. **Assumption Q-02:** 10 available hours/week, 140 hours over 14 weeks. Plan 112 focused hours and reserve 28 hours for discovery, defects and interruptions. Weekly effort includes tests and documentation, not only coding. If actual capacity is lower or unknown, re-estimate before committing a date.

This is an ambitious limited-pilot plan. The entire Phase 1 wish list is not credible as a fixed promise at this capacity. One validated sales source and one purchase path form the vertical slice. A second vendor, a second unvalidated container format and price alerts must not delay that slice.

- **P0 / pilot must-have:** membership and isolation, validated sales/purchase imports, duplicate/reconciliation controls, manual mappings, suppliers/ingredients, units, basic versioned recipes, explainable historical cost estimates, minimal summary/table, data-quality alerts, and recovery evidence.
- **P1 / full Phase 1 target, deferrable beyond week 14:** remaining CSV/XLSX support if not in the pilot path, second representative source only if justified, supplier price-change alerts and acknowledgement, dashboard enhancements.
- **P2 / later:** generalized compatibility, email/SMS, automatic matching, rich charts, multi-location/currency, subrecipes, billing automation and operational integrations. Proprietary POS belongs exclusively to Phase 2, not this backlog.

When P1 is cut, name the outcome a limited pilot and list missing Phase 1 features. Never reduce isolation, correctness or coverage disclosures to preserve scope.

## 2. Sequenced weekly backlog

Each row budgets 8 focused hours plus 2 reserve hours. IDs link to PRD feature acceptance; evidence in the exit column is required before calling work complete.

| Week | Priority / backlog item | Dependencies | Deliverable and measurable exit evidence |
|---|---|---|---|
| 1 | P0 B01: obtain samples and interview operator | Pilot access | One representative sales export and purchase export, totals references, documented row grain/units/tax/IDs; confirm actual weekly hours (Q-02). No broad importer implementation. |
| 2 | P0 B02: reconcile and freeze narrow contracts | B01 | G1: two periods and overlap cases manually reconciled; selected mappings and duplicate policy accepted; three recipes worked independently; unresolved tax/refund cases explicitly blocked. Optional second source only if samples already available. |
| 3 | P0 B03: minimal backend/frontend/database foundations and auth/isolation spike | G1, Q-07 decision | Selected tool versions and wrappers; CI skeleton; tenant membership and runtime DB/RLS prototype pass two-tenant tests, including reused connection. F01 baseline. |
| 4 | P0 B04: first sales import slice | B03, validated sales format | Upload/preview/commit one supported layout; counts/totals match fixture; exact reimport adds zero rows; malformed input commits none. F02/F10 core tests. |
| 5 | P0 B05: suppliers, ingredients and purchase import | B04, purchase mapping | Purchase pack conversion and normalized prices reconcile to sample; archive preserves history; duplicate purchase import is safe. F04/F06 foundations. |
| 6 | P0 B06: catalogue mapping and unit constraints | B05 | Unresolved items visible and manually resolvable; invalid dimensions rejected; cross-tenant linked IDs fail. One browser path from files to mapped records. F03/F06. |
| 7 | P0 B07: menu/recipe versions | B06, Q-05/Q-09 rules | Positive yields/components, nonoverlapping dates, immutable published recipe; three operator-checked recipes pass golden examples. F05. |
| 8 | P0 B08: historical costing and run snapshots | B07, purchase history | Price-date and recipe-date boundaries, weighted same-day prices, unknown costs and refund rules pass independent tests; stale run remains reproducible. F06/F07. |
| 9 | P0 B09: minimal analytics dashboard | B08 | Date filter, full net sales, known-subset profit/margin and coverage agree with golden dataset; incomplete/empty/stale states work; Playwright core journey passes. F07/F09. |
| 10 | P0 B10: stabilize core; conditional P1 price changes | B09, Q-10 if alerts selected | First review core defects and capacity. Only if P0 stable, normalized supplier comparison and deduplicated in-app alert tests pass. Otherwise spend all effort on P0. F08 is not presumed delivered. |
| 11 | P0 B11: edge cases and isolation hardening | B09 | Overlap/conflicting correction, upload limits, crash/retry and tenant-matrix tests pass. Add remaining CSV/XLSX path only if reserve/capacity permits after these gates. |
| 12 | P0 B12: deployment/recovery/performance rehearsal | B11, Q-08/Q-11 | Restore test dataset and reconcile; fresh/upgrade migrations pass; measure specified import/report targets on recorded machine; retention and access revocation checked. |
| 13 | P0 B13: supervised pilot | B12 and live-data gates | Operator completes full workflow on selected real exports; reconcile two periods, validate estimates and disclosures; record usability timing and unresolved issues. |
| 14 | P0 B14: release decision and contingency | B13 | Fix blockers; repeat affected gates; publish supported layouts/limits and deferred list. Release only with evidence, otherwise extend pilot preparation. No last-week feature expansion. |

No parallel workstreams are assumed. Security tests are developed with features, not deferred entirely to week 11. Reserves may absorb earlier overruns; do not repeatedly count the same reserve as feature capacity.

## 3. Decision gates and fallback plans

| Gate | Evidence | If not met |
|---|---|---|
| G1 / end week 2 | Source contracts and independent reconciliation; plausible ingredient mapping and recipes | Remain in discovery; drop second source/format and P1 forecast. Do not finalize broad compatibility or silently invent semantics. |
| G2 / end week 4 | Authenticated isolated tenant imports one sales layout safely | Re-estimate remaining P0 hours; cut all optional source/UI work. No live-data pilot before isolation passes. |
| G3 / end week 8 | Both datasets, recipes and explainable historical estimates work end to end | Move supplier price alerts and all enhancements beyond week 14. If minimum costing still infeasible, extend schedule rather than deliver misleading margins. |
| G4 / end week 12 | P0 tests, restore rehearsal and operational decisions complete | Delay live-data pilot. Synthetic demonstrations are allowed but are not a release. |
| G5 / week 14 | Operator reconciliation and no unresolved security/data-integrity blocker | Release limited pilot with explicit limitations, or continue validation with a revised plan. |

At each gate compare remaining available hours with estimated P0 work. If P0 exceeds capacity, cut P1 first, simplify UI, retain one format path per dataset, then extend the date if still necessary. Manual mapping/provisioning is acceptable; manual database repair required for routine imports is not a successful self-service pilot.

## 4. Prioritized risk/validation backlog

| Risk | Earliest mitigation / owner | Trigger for replanning |
|---|---|---|
| Export samples unavailable or inconsistent | Developer + pilot operator, B01/B02 | G1 cannot reconcile a complete sales/purchase path |
| Missing stable IDs and corrected exports | Developer, B02/B04 | Ambiguous overlap common enough to block routine usage; validate period replacement before expanding imports |
| Recipes, pack sizes, modifiers or prices incomplete | Operator + developer, B02/B06/B07 | Pilot results have insufficient useful cost coverage; fix inputs/scope before adding charts |
| Latest purchase price is misleading | Operator, B02/B08 | Worked historical examples do not reflect operator expectations; revise policy and tests explicitly |
| Tenant leakage or unsafe upload processing | Developer, B03 onward | Any reproducible isolation/integrity exploit blocks release |
| Auth/hosting/backup complexity consumes budget | Developer, B03/B12 | Infrastructure work displaces P0; simplify managed-service choices or extend date |
| Part-time capacity below 10 h/week | Developer, weekly | Remaining P0 estimate exceeds capacity; publish revised forecast |
| UI scope grows into POS or inventory | Product owner + developer, every change | Reject/reclassify into Phase 2 or separate later analytics scope |

## 5. Definition of pilot done

All P0 feature criteria in PRD pass within the documented supported layouts and volume envelope. JUnit, PostgreSQL integration/migration/isolation tests, frontend build/typecheck and critical Playwright flows pass. Two operator-reconciled periods and three recipe examples are accepted. No missing cost is disguised as zero; no unconfirmed rule is presented as an established fact. Recovery, access revocation and retention decisions are recorded before real data.

The developer records test commands, results, dataset/version and known gaps. Failed or skipped checks remain visible. Schedule completion alone is not completion of the product scope.
