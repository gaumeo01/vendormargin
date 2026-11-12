---
name: margin-calculator
description: Define, implement, or verify VendorMargin revenue, cost of goods sold, gross profit, gross margin, markup, and aggregate profitability calculations. Use for calculation rules and discrepancies, not file parsing or unrelated financial advice.
---

# Margin Calculator

## Scope and triggers

Use for requests to calculate product/vendor margins, explain mismatched profit totals, handle missing costs or refunds, or implement profitability reports.

Phase 1 analyzes imported POS exports. Phase 2 builds VendorMargin's own POS. Keep calculation rules independent of the sales source so both phases use consistent definitions. Do not add Phase 2 sales entry or payment behavior as part of analytics work.

## Workflow

1. Inspect existing definitions, tests, normalized sales fields, and cost records. Identify the requested reporting grain and period, currency, timezone, and product/vendor attribution rules. Distinguish established business rules from assumptions.
2. Establish the revenue and cost basis before calculating. Determine whether source sales already include discounts, refunds, or tax to avoid subtracting them twice. For gross profit, normally use sales after discounts and returns, excluding collected sales tax; reconcile any repository-specific definition explicitly. Do not describe gross profit as net profit.
3. Select the cost basis from established rules: historical sale-time cost, effective-dated purchase cost, weighted average, or another documented method. Never substitute today's cost for historical cost silently. Keep unknown costs distinct from zero; report incomplete cost coverage and do not present partial profit as complete. Align revenue and cost populations when reporting a known-cost subset.
4. Use these definitions on a consistent basis:
   - Gross profit = net revenue - cost of goods sold (COGS).
   - Gross margin percent = gross profit / net revenue * 100.
   - Markup percent = gross profit / COGS * 100.
   A zero denominator produces an explicitly unavailable percentage, not infinity or a fabricated zero. Explain negative-revenue periods rather than silently clamping results.
5. Define refund and void treatment. A void contributes no completed sale. Refunds reverse the appropriate revenue; cost reversal depends on the documented return/restock rule and may differ for partial refunds. Avoid reversing both signed source values and a refund flag a second time.
6. Calculate with decimal values (BigDecimal in Java) or a documented fixed-point representation. Specify scale, rounding mode, and rounding stage; support fractional quantities. Allocate order-level discounts deterministically, preserving the original total including rounding remainders.
7. Aggregate revenue and COGS first, then calculate the percentage; do not average row percentages. Keep currencies separate unless an explicit exchange-rate policy is supplied. Preserve traceability to sales and cost inputs, and disclose missing costs, excluded rows, and unresolved assumptions with results.

## Testing requirements

Use independently worked examples for implementation changes:

- Revenue 100 and COGS 60 yield profit 40, margin 40%, and markup approximately 66.67% at two decimal places.
- Rows (revenue 100, cost 50) and (revenue 900, cost 810) yield total profit 140 and margin 14%, not the 30% average of row margins.
- Zero revenue, zero cost, unknown cost, losses, negative net revenue, and fractional quantities follow explicit availability and sign rules.
- Discounts, inclusive/exclusive tax, voids, full and partial refunds, and return-cost rules do not double count adjustments.
- Historical cost boundaries, currency separation, rounding ties, and discount allocation remainders behave as specified.
- Aggregated revenue, COGS, and profit reconcile to included rows; report filters and date boundaries select the expected population.

Run focused calculation tests and report/query integration tests when those layers change. State tested rounding and cost assumptions and any gaps. For design-only requests, provide rules and worked examples without implementing application code.