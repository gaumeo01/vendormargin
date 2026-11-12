---
name: pos-data-importer
description: Design, implement, or debug VendorMargin POS export imports, column mappings, normalization, validation, reconciliation, and duplicate handling. Use for external POS file ingestion, not checkout or payment processing.
---

# POS Data Importer

## Scope and triggers

Use for requests such as adding a POS export format, fixing incorrect imported totals, mapping vendor columns, or making reimports safe.

VendorMargin Phase 1 imports and analyzes exports from existing POS systems. Phase 2 builds our own POS. Keep vendor-specific parsing separate from normalized sales data so future native sales can use the same analysis contracts. Do not build checkout, payments, or inventory operations merely to support an import.

## Workflow

1. Read repository guidance and inspect existing import contracts, fixtures, and persistence before changing them. When absent, propose the smallest contract needed for the requested format; do not assume a POS vendor, database, or file schema.
2. Inspect representative, sanitized export samples. Identify row grain (sale, line item, tender, or summary), headers, encoding, delimiter, date format, timezone, currency, decimal separators, and tax/discount conventions. Ask for missing semantics when they affect correctness; do not infer them silently from ambiguous values.
3. Define explicit source-to-domain mappings. Preserve identifiers as strings, including leading zeros. Keep source file/batch identity, source row, and mapping version or equivalent provenance so normalized values can be traced back. Do not invent missing costs or product matches.
4. Separate parsing, normalization, validation, and persistence. Distinguish blanks from zero, voids from refunds, transaction totals from item totals, and gross amounts from net amounts. Avoid double counting mixed detail and summary rows. Use decimal monetary arithmetic and explicit currency and date handling.
5. Define duplicate identity using source system and available stable sale/line identifiers within the relevant business scope. File hashes can detect exact repeats but do not alone detect overlapping exports. Specify how corrected records differ from duplicates; flag ambiguous matches instead of silently merging them.
6. Validate before committing. Provide accepted, rejected, and duplicate counts plus actionable row/column errors. Make atomic versus partial acceptance explicit and ensure retries cannot duplicate accepted rows. Preserve source data without exposing sensitive row contents in logs.
7. Reconcile normalized counts and totals against comparable source totals, accounting for exclusions and documented rounding. Surface unexplained differences. Hand off normalized facts to margin calculations rather than embedding vendor-specific margin formulas in parsers.

## Testing requirements

When implementation is requested, use sanitized or synthetic fixtures with independently determined expected outputs:

- A representative valid file and relevant format variations, including quoted delimiters, blank values, identifier leading zeros, localized numbers, and ambiguous dates.
- Missing required columns, invalid rows, empty files, unsupported formats, and clear rejection diagnostics.
- Discounts, tax-inclusive/exclusive values, refunds, voids, and mixed summary/detail rows when supported by the format.
- Exact reimports, overlapping exports, corrected records, and interrupted/retried persistence; verify observable stored totals and duplicate counts.
- Reconciliation of accepted, rejected, and duplicate rows and source totals. Test transaction rollback or partial acceptance according to the chosen contract.

Run focused parser tests and persistence integration tests for affected behavior. Report commands, results, unresolved mappings, and any untested formats. For instructions or design-only requests, review mappings and test cases without scaffolding application code.