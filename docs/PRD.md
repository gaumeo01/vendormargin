# VendorMargin MVP Product Requirements

Status: draft for validation. Documentation only; no application has been implemented.

## 1. Authority, scope, and terminology

**Confirmed requirements:** VendorMargin serves cafes and small restaurants. Phase 1 analyzes sales and purchase exports from existing systems plus user-defined ingredient recipes. It includes suppliers, ingredients, unit conversion, estimated gross margins, supplier price changes, dashboards/alerts, and secure tenant isolation. The stack is Java 21, Spring Boot, PostgreSQL, Flyway, React, TypeScript, Vite, Tailwind CSS, JUnit, Testcontainers, Playwright, Docker Compose, and GitHub Actions. Delivery planning assumes one part-time developer over 14 weeks.

**Phase boundary:** Phase 1 has no proprietary POS, order entry, checkout, payments, receipts, cash drawers, stock movements, or live POS integration. Purchase history is evidence for estimated ingredient costs, not an inventory ledger. Phase 2 may supply native sales to the same analytics contracts; no Phase 2 functionality is built now.

Everything marked **A-xx** is a proposed assumption, not an approved business or technical decision. **Q-xx** identifies a decision requiring validation. Feature priorities, acceptance thresholds, schema, and architecture below are proposals. Approval to author these documents does not approve every default. Record decisions here with evidence before treating assumptions as settled.

## 2. Problem, users, and outcome

Owners need to understand which menu items may have weak ingredient margins and which supplier prices have increased, without replacing their POS. Their exports and recipes can be inconsistent or incomplete. The product must make data quality visible rather than turn incomplete inputs into apparently precise profit figures.

Proposed primary user: an owner/operator who exports files, maps products, enters recipes, and reviews weekly results. A read-only accountant/manager role is optional after the owner workflow works. The pilot should prove a repeatable path from real exports to explainable estimates before expanding compatibility.

### Proposed success measures

- At least one pilot operator completes import -> mapping -> recipe -> margin review without developer edits to their data.
- Two independently reconciled sample periods from the selected source produce the agreed net-sales totals; any difference is itemized and explained.
- Every displayed estimate can be traced to sale rows, recipe version, ingredient prices, and calculation policy.
- All automated cross-tenant negative tests pass; no unresolved isolation defect is acceptable for release.
- A pilot user can identify why a margin is unavailable or why an ingredient price changed using the UI alone.
- Proposed usability target: an already configured source's next valid export reaches results within 10 minutes of user effort. Measure during the pilot; this is not a promised SLA.

## 3. Validation before broad compatibility

**Gate G1, weeks 1-2:** obtain sanitized exports for one POS source, and optionally a second only if capacity allows. Obtain purchase samples too; they may originate from supplier/accounting exports rather than the sales POS. Do not promise a universal importer, a vendor catalogue, or automatic schema inference.

For each source collect two overlapping periods, a corrected export if available, a totals report, and examples of discounts, taxes, voids and refunds. Record row grain, identifiers, unit/currency conventions, business dates, totals definitions, and export version. Use synthetic edge cases where real samples are unavailable, labeled as synthetic.

G1 passes when an operator confirms a mapping and independently calculated reconciliation for at least one sales and one purchase path, with an explicit duplicate/correction strategy. If this evidence is missing, continue discovery and reduce later scope; do not generalize from invented exports. The schema in [DATABASE.md](DATABASE.md) is provisional until this gate passes.

CSV and Excel remain Phase 1 objectives. **A-01:** initially support CSV and non-macro `.xlsx` for validated layouts. Legacy `.xls`, password-protected workbooks, formulas requiring evaluation, and arbitrary workbook layouts are unsupported. If only one container format is proven within the schedule, call the release a limited pilot and document the unsupported format; do not claim CSV/Excel completeness.

## 4. Proposed release scope and feature acceptance

P0 is required for a trustworthy limited pilot. P1 is the Phase 1 MVP target but may move beyond week 14. P2 is deferred. Test expectations apply when features are implemented, not to this documentation task.

| ID / priority | Feature and user outcome | Measurable acceptance criteria | Required verification |
|---|---|---|---|
| F01 / P0 | Authenticated tenant workspace | Only active members can access a tenant; absent/invalid membership fails; every tenant-owned operation checks scope. A user-supplied resource ID cannot reveal another tenant's data or existence. | Integration tests with two tenants and an unauthenticated client across reads, writes, imports, status, analytics and artifacts; test connection reuse and background execution. |
| F02 / P0 core; P1 second format/source | Import sales and purchases | One validated path for each dataset; preview shows source version, period, counts and totals. Unsupported headers/types produce actionable errors. Reimport adds zero duplicate facts. No unexplained reconciliation difference. | Golden CSV/XLSX fixtures for each supported path; malformed, empty, over-limit, localized numeric/date, leading-zero ID, overlapping and corrected exports; concurrent retry/rollback tests. |
| F03 / P0 | Resolve source mappings | Each imported sale product maps to one menu item or is visibly unresolved; each purchase item maps to one ingredient/pack conversion or is unresolved. Ambiguous matches never silently merge. Changing mappings invalidates affected estimates. | Integration tests for collisions, unresolved rows and remapping; Playwright mapping-to-result journey. |
| F04 / P0 | Manage suppliers and ingredients | Create/edit/archive without deleting historical references. Positive pack sizes and compatible units required. Archived records remain usable in historical reports. | Validation and tenant-ownership tests; archive/history integration tests. |
| F05 / P0 | Menu items and recipes | Publish a recipe with positive yield and ingredient amounts; a menu item has at most one effective recipe for a date. Published versions are immutable. Uncostable recipes show ingredient-level reasons. | Unit/integration tests for overlapping dates, missing prices, recipe edits, yield and archive behavior; one complete recipe UI journey. |
| F06 / P0 | Conversion and estimated cost | Same-dimension conversions preserve quantities; incompatible conversions fail. Supported recipes and historical prices match independent worked examples at documented precision. No unknown cost is silently zero. | JUnit calculation fixtures covering pack factors, decimal quantities, yield, effective-date boundaries and rounding; PostgreSQL numeric persistence tests. |
| F07 / P0 | Estimated gross margin | Date-filtered summary and menu table show net sales, estimated ingredient cost/profit, margin and cost coverage. Aggregate percentage is derived from sums. Zero denominator returns unavailable. | Golden multi-item/multi-date dataset, missing-cost subset, refund/void tests, date-boundary tests, query integration and Playwright smoke test. |
| F08 / P1 | Supplier price increases | Comparable normalized prices yield exact absolute/percentage changes. First/zero-price baseline shows unavailable percentage. One visible alert per qualifying comparison and policy version; retries do not duplicate it. | Tests for changed pack size, supplier separation, equality at threshold, price decreases, credits, late imports and repeat evaluation. |
| F09 / P0 minimal; P1 enhancements | Dashboard and alerts | Summary/table reflect identical filters and snapshot; data-quality warnings link to unresolved records. Empty/loading/error states present. P1 adds price alerts and acknowledgement; charts and notifications are optional. | Playwright import -> recipe -> dashboard journey, empty/error states, basic keyboard navigation and narrow viewport check; independent aggregate reconciliation. |
| F10 / P0 | Operability and recovery | Import interruption creates no partial committed facts; errors correlate to a batch ID without raw sensitive rows. Backup restore reproduces a test tenant's counts and totals. | Crash/retry integration tests, restore rehearsal, migration upgrade test, health endpoint and sanitized-log review. |

P2: generic mapping designer, automatic fuzzy matching, second POS vendor unless trivial, scheduled imports, email/SMS alerts, advanced charts, forecasts, nested recipes, inventory/waste tracking, multi-currency conversion, multi-location rollups, subscriptions/billing automation, and enterprise administration. Pilot provisioning may be manual through controlled administrative procedures; not unprotected APIs.

## 5. Business rules and calculation assumptions

### Sales, purchases and provenance

**A-02:** one location, reporting currency and IANA business timezone per tenant for the pilot. Do not default these from the developer's machine. Mixed currencies or locations are rejected or require an explicitly supported mapping before import. Store original timestamps and normalized business dates where supplied; date-only rows do not acquire fabricated event times.

The adapter defines whether a source amount already includes tax, discounts or refunds. Net revenue is sales after discounts and returns, excluding collected sales tax under the proposed policy. Never subtract an adjustment twice. Voids contribute no completed sale. Detail and summary rows cannot both contribute to totals. Daily menu aggregates are acceptable only if that source's grain and overlap policy are validated; they must not coexist with overlapping transaction detail.

**A-03:** stage and validate first; commit each batch atomically. Invalid financial rows block commit. Unresolved catalogue mappings may commit as explicit unknowns but cannot become fully costed estimates. Report total rows = candidate rows + duplicate rows + rejected rows + intentionally excluded rows, using mutually exclusive categories. At commit, accepted rows equal candidates; a blocked batch accepts zero. Display reasons for exclusions and reconcile comparable monetary totals separately.

Stable source IDs are preferred for deduplication. Scope identities by tenant, source and dataset. Exact file hashes detect only exact repeats. If stable identities are absent, the format must define an operator-confirmed period/grain policy; reject ambiguous overlap rather than guess. Conflicting corrections are blocked in the first pilot; automatic upserts and replacement imports are deferred until a correction policy is validated. This is a usability risk, not a claim that corrected exports are supported.

### Units, recipes and historical pricing

**A-04:** base units are grams, millilitres and each. Convert kg/g and L/ml within dimensions. A supplier pack needs an explicit positive amount in an ingredient base unit. Do not convert mass to volume without ingredient-specific density, or each to mass without a defined factor. Such cross-dimension mappings are unsupported initially. Fractional counts are permitted only for ingredients explicitly configured to allow them.

**A-05:** recipe quantities are as-purchased inputs for an explicit positive number of menu servings. Ingredient preparation loss is already reflected in input quantities; no additional waste percentage is applied. Subrecipes, variable portions and modifier-specific recipes are deferred. Unsupported modifiers must be disclosed and affected lines marked uncostable, not assigned the base recipe silently.

**A-06:** ingredient unit cost uses the latest valid purchase observation on or before the sale's business date across suppliers for that ingredient. Use purchase effective date, never import time. If observations share that date without a meaningful order, use their quantity-weighted unit cost. This is a practical replacement-cost proxy, not FIFO, stock valuation, or actual COGS. No prior price means unknown, not use of a future price. Exclude credit notes/returns from the price baseline until their semantics are validated.

Proposed normalized purchase unit cost = eligible line amount / normalized positive quantity. Eligible line amount is after line/allocated discounts and excludes recoverable tax. **Q-04:** confirm recoverable versus nonrecoverable tax and freight treatment with the pilot operator; until resolved, mark cost basis unverified. Proposed initial freight treatment is exclusion with disclosure, not an accounting assertion. Order discounts require deterministic proportional allocation and stable rounding-remainder assignment; block ambiguous allocation inputs.

Recipe cost per serving = sum(ingredient input quantity in base units * applicable ingredient unit cost) / recipe yield in servings. Estimated ingredient cost for a sale = fulfilled quantity * recipe cost per serving. Use the recipe version effective on the sale date. Historical corrections create new versions with explicit supersession; published contents are never edited in place.

**A-07:** cash refunds lower revenue but do not automatically recover consumed ingredients. Retain original ingredient cost for already fulfilled cafe sales. If source data establishes cancellation before preparation, exclude that quantity/cost. If refund rows cannot be related to fulfilment, affected estimates are unavailable until an explicit source rule is agreed. Negative signed rows and refund flags must not cause double reversal.

A linked cash-refund row adds negative revenue and no additional ingredient cost; the original fulfilled sale retains its cost on its original business date. A refund in a later reporting period therefore does not move or repeat that original cost. Test same-period and cross-period refunds separately. Cancellation corrections and unidentified refund fulfilment remain blocked or explicitly uncostable rather than silently adopting this linked-refund rule.

### Margins, completeness and precision

- Estimated ingredient gross profit = net revenue - estimated ingredient cost.
- Estimated ingredient gross margin % = estimated ingredient gross profit / net revenue * 100.
- Markup, if later exposed, uses cost as denominator; it is not gross margin.
- Aggregate numerator and denominator first. Never average menu-item margin percentages.
- Zero revenue yields unavailable margin. Negative net revenue retains its sign and a warning; do not clamp or imply ordinary profitability.
- Missing any required ingredient price or recipe makes that sale line's estimated cost unknown. Display full net sales separately from revenue/profit/margin for the known-cost subset.
- **A-08:** coverage = known-cost non-void sale-line count / eligible non-void sale-line count; show both counts. No eligible rows means unavailable, not 100%. Supplement with excluded/unresolved net revenue; count coverage alone can hide high-value gaps. Refund limitations remain visible separately.
- **A-09:** BigDecimal/PostgreSQL numeric arithmetic, unit/quantity precision up to 6 decimals, working cost precision up to 10 decimals, HALF_UP rounding at persisted calculation outputs. Currency amounts display at configured currency precision; percentages at two decimals. Source precision beyond supported storage is rejected, never silently truncated. Sum unrounded working values before rounding report totals. Confirm this policy with fixtures before implementation.

Worked examples, under the proposed rules:

1. Purchase 1 kg coffee for 20 currency units -> 0.02/g; 18 g serving costs 0.36. Ten fulfilled servings at net revenue 30 produce estimated profit 26.40 and margin 88%, excluding other ingredients in this deliberately single-ingredient example.
2. Revenue/cost pairs (100, 50) and (900, 810) -> revenue 1,000, profit 140, margin 14%; the average of row margins, 30%, is incorrect.
3. Two sale lines have revenue 100 each; only one has known cost 60. Full revenue is 200, coverage 1/2 (50%), and known-subset profit 40/margin 40%. Do not report full profit 140.
4. A fulfilled serving sold for 10 with ingredient cost 3 is fully refunded. Revenue becomes 0, cost remains 3, estimated profit -3 and margin unavailable under A-07.

### Supplier price changes

**A-10:** compare a supplier/ingredient's newest dated, normalized positive purchase price with its preceding comparable dated price in the same currency and tax basis. Aggregate same-day observations by quantity weighting. Percent change = (new - previous) / previous * 100. Different pack sizes compare after normalization. No prior price or prior zero price means unavailable percentage.

Proposed default threshold is strictly greater than 5%; configuration and threshold require pilot validation. Late imports recompute chronological comparisons; superseded alerts are not presented as current. Start with a dashboard badge/list, not email/SMS or real-time delivery guarantees.

### Recalculation

**A-11:** calculations are explicit immutable runs over committed input versions. New imports, mappings, recipes or price corrections mark reports stale. Recalculate on user request initially; no scheduler required. Keep run timestamp, input revision, policy version, missing-cost reasons and cost lineage. Reports never mix rows from different runs. Preserve prior runs subject to an agreed retention policy; label current recomputed estimates versus previously viewed estimates.

## 6. Nonfunctional acceptance proposals

**A-12:** initial envelope: 10 MiB upload, 10,000 data rows/file, 100,000 sales rows/tenant, and one active import or calculation per tenant. Validate these against samples and available hardware in weeks 1-2. Bound XLSX decompressed size/cells and parsing time, not only compressed upload size. Never evaluate workbook formulas or execute macros. Unsupported types fail before commit.

On a recorded reference machine and 100,000-row test tenant, propose dashboard API p95 <= 2 seconds across 30 warm requests and a 10,000-row import completing within 60 seconds. These are test targets subject to measurement, not availability commitments. Repeated requests must not change results or cross tenant boundaries.

Authenticated access, secure credential handling, bounded uploads and tenant isolation are non-negotiable outcomes. Proposed implementation mechanisms are in [ARCHITECTURE.md](ARCHITECTURE.md). Hosting, auth provider, retention and recovery objectives remain open. No real pilot data enters an environment before access, backup, deletion and retention decisions are validated.

## 7. Assumptions and open decisions register

| ID | Open decision / assumption to validate | Evidence and decision deadline | Consequence if unresolved |
|---|---|---|---|
| Q-01 | POS vendors, sample access, row grain and formats (A-01) | Operator walkthrough and paired exports; week 2 | No importer compatibility commitment; G1 blocked |
| Q-02 | Part-time availability; planning basis 10 h/week | Developer availability; week 1 | Replan hours and cut P1 before promising date |
| Q-03 | One location/currency, timezone, owner-only access (A-02) | Pilot scope agreement; week 2 | Avoid multi-location schema/UI promises |
| Q-04 | Revenue tax basis, purchase tax/freight, refunds (A-07) | Annotated sample reconciliation; week 2 | Financial estimates remain unverified |
| Q-05 | Recipe yields/modifiers and price policy (A-04 through A-06) | Three operator-checked recipes and historical examples; week 2 | Missing coverage; costing design cannot finalize |
| Q-06 | Duplicate/overlap/correction semantics (A-03) | Reimports and corrected sample; week 2 | Reject ambiguous files; may block viable pilot |
| Q-07 | Authentication/provider, membership provisioning, deployment | Small implementation decision/spike; week 3 | No public access or real-data pilot |
| Q-08 | Data limits, expected volumes, retention/raw-file storage | Samples, operator needs, deployment budget; weeks 2-3 | A-12 unproven; no performance claims |
| Q-09 | Precision, coverage and recalculation policy (A-08/09/11) | Golden examples and operator sign-off; week 7 | Cannot call estimates accepted |
| Q-10 | Price alert comparison and threshold (A-10) | Operator review of historical price changes; week 10 | Defer alerts rather than present misleading ones |
| Q-11 | Backups, restore objectives, tenant deletion and access revocation | Hosting decision plus rehearsal; before pilot | Real-data pilot blocked |

The developer maintains this register; the product owner/pilot operator validates business semantics. If the same person fills both roles, explicitly record which evidence informed the decision. Do not infer confirmation from silence.

## 8. Major risks and release conditions

Highest risks are missing export samples, inconsistent source identities, purchase-to-ingredient matching, misleading margins from incomplete recipes, ambiguous historical prices/refunds, and insufficient part-time capacity. Tenant leakage and unsafe file parsing are release blockers regardless of schedule. Spreadsheet complexity and connector proliferation can consume the entire budget; validate narrow paths first.

A limited pilot may omit P1 with visible limitations. It cannot omit authentication/isolation, import reconciliation, truthful cost coverage, deterministic calculations, or recovery checks. Full Phase 1 feature completion is a separate milestone if scope slips. Use [ROADMAP.md](ROADMAP.md) for week-by-week gates and reductions; [DATABASE.md](DATABASE.md) and [ARCHITECTURE.md](ARCHITECTURE.md) remain proposed designs until the relevant questions are resolved.
