# Product Return Analysis — PT Medika Sejahtera

SQL-based exploratory analysis of 59 product return cases from a medical supplies distributor in 2024, aimed at identifying the operational root causes behind returns and quantifying their financial impact.

> **Note:** The dataset used in this project is synthetic, created to simulate a realistic business scenario. It builds on the same PT Medika Sejahtera dataset used in an earlier project, [Analysis of Delivery Note Return Delays](https://github.com/Rpfitrah/dn-analysis-medika-sejahtera), which looked at a different operational problem (delayed document returns) for the same company.

## Business Context

PT Medika Sejahtera distributes medical products (diagnostics, lab supplies, surgical items, consumables) to hospitals and clinics. Every return costs the company money and adds operational overhead, but returns were being logged without a clear picture of *why* they were happening or *where* to focus fixes. This analysis was commissioned to turn a year of raw return records into a prioritized, evidence-backed action list.

## Business Questions

1. What reason code most frequently causes product returns?
2. Which products and product categories are returned most often?
3. What is the total financial loss from returns in 2024, and which products/categories drive it?
4. Which customer type returns the most?
5. Which region has the highest number of return cases?
6. Is there a monthly or seasonal pattern in returns across the year?

## Data & Tools

- **Data:** 59 return records for 2024 (`return_cleaned`), cross-referenced with invoice-level data (`invoice_data`, `invoice_detail`) and return quantity detail (`return_data`) to compute rates and unit-level breakdowns.
- **Database:** PostgreSQL
- **Tools:** SQL (CTEs, window functions — `SUM() OVER (PARTITION BY ...)` for share-of-group percentages across nearly every breakdown, `LAG()` for quarter-over-quarter growth), Jupyter Notebook via `ipython-sql`, Python/pandas/SQLAlchemy for the connection layer.
- Full query set and inline analysis notes: [`EDA.ipynb`](./EDA.ipynb)

## Process

The analysis follows a screen → verify → conclude structure rather than jumping straight to conclusions:

1. **1D screening** — established a baseline distribution for each dimension (reason code, region, quarter, category, customer type, product), then flagged any subgroup that deviated meaningfully from that baseline as a candidate worth investigating further.
2. **Confounding checks** — when two flagged subgroups shared suspiciously similar numbers (e.g., a region and a customer type with an identical case count), a cross-tab was run before treating them as separate findings, to rule out double-counting the same signal.
3. **Rate normalization** — raw counts were converted into rates against invoice volume per quarter, since a rising return count means little without knowing whether transaction volume grew at the same pace.
4. **Outlier drill-down** — for categories with disproportionate financial loss, individual return lines were pulled and joined against invoice/return-quantity detail to identify the specific transactions responsible, rather than treating the loss as evenly spread across the category.
5. Sample sizes were kept in view throughout — several sub-breakdowns (e.g., single-digit counts at the product/SKU level) were explicitly excluded from conclusions once cell sizes got too small to be reliable.

## Key Findings

- **Returns are driven by process, not product.** Two reason codes — Order Entry Error  (SIS, 47.46%) and Customer Order Error (SPC, 33.90%) — account for 81% of all cases. Product category distribution is fairly even (8–16 cases each), which rules out a specific product or product line as the root cause.
- **SIS and SPC follow different patterns, so they need different fixes.** SIS behaves like a one-time spike, jumping to 66.67% of cases in Q2 alone — a 336% surge in its rate against invoice volume, far outpacing the 14.8% growth in transactions over the same period. SPC instead shows a steadily worsening trend, climbing every quarter (1.34% → 1.75% → 2.23% → 5.39% of transactions), with growth (141% Q3→Q4) again far outpacing transaction volume growth (14%).
- **Financial loss is far more concentrated than case count suggests.** Of the total IDR 171,506,500 in losses, 45% comes from just 5 cases (8.5% of all returns) — all of them large-quantity orders (24–58 units), and 4 of the 5 already tied to SIS or SPC. Surgery & Procedures is the most disproportionately costly category (31% of loss from 19% of cases), driven by per-case value rather than volume.
- **Private Hospital and West Jakarta lead in volume for different reasons.** Private Hospital has the most returns (45.76%) but its reason-code mix matches the overall baseline closely — its lead is most likely just a function of being the largest customer segment, not a process problem. West Jakarta, by contrast, shows a genuine SIS concentration (61.54%, +14 pts above baseline), confirmed independent of the Government Hospital finding via a cross-tab check.

## Recommendations

1. Add a quantity-based approval step for large orders, applied across all categories — the loss data shows the risk is tied to order size, not category, so a category-specific control (e.g., Diagnostics-only QC) would miss cases in categories like Consumables that look low-risk in aggregate.
2. Investigate staffing or process changes around Q2, since the SIS spike's timing suggests an internal trigger that the current dataset can't identify directly.
3. Review the manual PO intake/verification process, the most likely source of the worsening SPC trend.
4. Prioritize West Jakarta for a system-input process audit, given its confirmed, independent SIS concentration.

## Limitations

- Several breakdowns (individual regions, products, and Q1 reason-code cells) have single-digit sample sizes and were intentionally excluded from the conclusions above — they're noted in the full analysis as directional only.
- The dataset doesn't include total transaction volume per customer type, so Private Hospital's volume lead is inferred to be segment-size-driven rather than confirmed directly.
- This is an exploratory, correlational analysis based on one year of historical data — it identifies *where* and *when* the problem concentrates, not a statistically confirmed root cause. The specific trigger behind the Q2 SIS spike, for instance, remains unidentified and would need further investigation (system logs, staffing records) beyond what this dataset provides.

## Repository Contents

- `EDA.ipynb` — full query-by-query analysis, with all markdown cells (screening notes, findings, conclusion) in English