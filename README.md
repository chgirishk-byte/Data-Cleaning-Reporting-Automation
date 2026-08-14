# Data Cleaning & Reporting Automation

An automated Python pipeline that takes a messy retail transactions dataset,
cleans it with a logged and reusable process, and generates a self-contained
visual HTML report — end to end in under a second.

## What this project demonstrates

- **Data preprocessing**: handling missing values, duplicates, and inconsistent
  categorical/text/date data with documented, defensible strategies
- **Automation**: a one-command pipeline (`main.py`) that chains data
  generation → cleaning → reporting, with every cleaning decision logged to JSON
- **Reporting efficiency**: matplotlib charts auto-embedded into a styled HTML
  report, no manual copy-pasting into slides or docs

## Problem simulated

Real transactional data is rarely clean. This project generates a synthetic
1,900+ row retail dataset with realistic messiness injected on purpose:

| Issue | Example |
|---|---|
| Missing values | ~150 blank `CustomerEmail`, `Region`, `Quantity` cells |
| Exact duplicates | 120 duplicate order rows |
| Inconsistent categories | `Electronics`, `electronics`, `ELECTRONICS`, `Electroniks` |
| Inconsistent regions | `Bangalore`, `bangalore`, `BLR`, `Bengaluru ` |
| Mixed date formats | `2025-05-16`, `26 Apr 2025`, `06-04-2025` |
| Invalid numerics | negative quantities, ₹0 unit prices, 100x fat-finger totals |
| Stray whitespace | `"  Denim Jacket  "` |

## Pipeline

```
scripts/generate_sample_data.py   → data/raw/retail_transactions_raw.csv
scripts/data_cleaning.py          → data/cleaned/retail_transactions_clean.csv
                                   → reports/cleaning_log.json
scripts/report_generator.py       → reports/data_quality_report.html
                                   → reports/charts/*.png (embedded in report)
scripts/main.py                   → runs all of the above in one command
```

### Cleaning strategy (logged at every step)

1. **Text standardization** — trim whitespace on all text columns
2. **Category / Region normalization** — keyword and lookup-based mapping to a
   single canonical spelling/casing per value
3. **Date parsing** — mixed-format strings parsed into a single `datetime` dtype
4. **Numeric error correction** — negative quantities converted to absolute
   value, ₹0 prices flagged as missing, totals inconsistent with
   `Quantity × UnitPrice × (1 - Discount)` recalculated
5. **Missing value imputation** — column-appropriate strategy, not a blanket
   drop/fill: median for `Quantity`, category-median for `UnitPrice`, business
   rule (0) for `Discount_%`, mode for `Region`/`PaymentMethod`, explicit
   `"unknown@unknown.com"` placeholder for `CustomerEmail`
6. **Deduplication** — dropped on all business fields except `OrderID`, since
   duplicate order entries can carry different IDs

Every decision is recorded in `reports/cleaning_log.json` so the transformation
from raw to clean is fully auditable — not a black box.

## Report output

`reports/data_quality_report.html` includes:
- KPI summary (orders, revenue, avg. order value, rows removed)
- Before/after cleaning charts (missing values, row counts)
- Business insight charts (revenue by category, revenue by region, monthly
  trend, payment method mix)
- Top 5 products by revenue table

Open it directly in a browser — no server or dependencies needed, all charts
are embedded as base64 images.

## How to run

```bash
pip install pandas numpy matplotlib

# Full pipeline, one command:
python scripts/main.py

# Or regenerate a fresh messy sample dataset first:
python scripts/main.py --regenerate
```

Or step through interactively in `notebook.ipynb`.

## Project structure

```
data-cleaning-automation/
├── data/
│   ├── raw/retail_transactions_raw.csv
│   └── cleaned/retail_transactions_clean.csv
├── reports/
│   ├── cleaning_log.json
│   └── data_quality_report.html
├── scripts/
│   ├── generate_sample_data.py
│   ├── data_cleaning.py
│   ├── report_generator.py
│   └── main.py
├── notebook.ipynb
└── README.md
```

## Key takeaways

- Missing-value strategy should be **column-specific**, not global — median for
  skewed counts, category-median for price, mode for categorical, and explicit
  "unknown" placeholders where imputation would be misleading (email addresses)
- Deduplication logic needs to match on **business meaning**, not just row
  equality — two rows with the same product/customer/date but different
  auto-incremented IDs are still duplicates
- Logging every cleaning step (not just the final output) is what turns a
  cleaning script into an **auditable pipeline** a stakeholder can trust
- Recomputing derived fields (`TotalAmount`) after correcting their inputs
  catches inconsistencies that a simple missing-value check would miss
