# Cleaning Decisions: `dirty_cafe_sales.csv`

> **Source:** https://www.kaggle.com/datasets/omniam0hamed/dirty-cafe-sales, file `dirty_cafe_sales.csv`
> **Output:** `output.csv`

## Summary

A cafe sales dataset of **10,000 transactions** (year 2023) with 8 columns, filled with
`ERROR`, `UNKNOWN` and empty values. The goal was to clean it **without guessing**:
a value is filled only when it can be derived from other columns of the same row;
otherwise it stays unknown and is marked with a flag column.

**Result:** 10,000 rows kept, 3 flag columns added (`Item_missing`, `Date_missing`,
`Numeric_missing`), and `Total Spent = Quantity × Price Per Unit` holds in 100% of
the rows that have all three values.

| What | Before | After |
|---|---|---|
| Rows | 10,000 | 10,000 (none dropped) |
| Item unknown | 333 empty + `ERROR`/`UNKNOWN` | 480 (prices 3.0 and 4.0 are ambiguous) |
| Rows with a missing Quantity / Price / Total | 138 / 179 / 173 empty + `ERROR`/`UNKNOWN` | 26 rows |
| Payment Method unknown | 5 variants (`ERROR`, `UNKNOWN`, empty...) | 3,178 (31.8%), merged into `unknown` |
| Location unknown | 4 variants | 3,961 (39.6%), merged into `unknown` |
| Transaction Date unknown | `ERROR`, `UNKNOWN`, empty | 460 (4.6%), `NaT` |

**Key decisions**
- Missing numbers are derived from the same row (`Total = Quantity × Price`), never estimated.
- `Item` is inferred from the price only when it identifies one item (1.0, 1.5, 2.0, 5.0).
- Payment Method and Location are not filled: no relationship in the data could predict them.
- Dates are not estimated: the transaction ID has no relationship with the date.

**Use with care:** analysis by payment method or location covers only about 60-68% of
the rows. State the number of rows used next to any such result.

---

## Principle: no guessing

A value is filled only when it can be derived from other columns **of the same row**.
Otherwise it stays unknown and is marked with a True/False flag column.

Why: an estimated value (mean, most frequent, etc.) looks like real data in later
analysis and cannot be told apart from it. An honest gap, with a flag, can be
filtered out or reported.

---

## Cleaning steps (in the order they were applied)

> **The order matters.** The price is filled from the item's median, and the item is
> inferred from the price. This is not circular, because each item has one fixed price
> and the items that are already known are used first.

### Step 1. `Transaction ID`
- All 10,000 values match `TXN_` + 7 digits, and all are unique.
- **Decision:** no changes needed.

### Step 2. Numeric columns: `Quantity`, `Price Per Unit`, `Total Spent`
- The raw columns contained the text values `ERROR` and `UNKNOWN`, so pandas read them as strings.
- **Decision:** replace `ERROR` / `UNKNOWN` with `NaN`, then convert to `float`.
- **Why `NaN` and not "unknown":** these columns must stay numeric so `sum` and `mean` work.

### Step 3. Fill `Quantity`
The business rule is `Total Spent = Quantity × Price Per Unit`, and it holds in **100%** of the rows where all three values exist (0 mismatches). So a missing `Quantity` is derived as `Total Spent ÷ Price Per Unit` where both are known.

### Step 4. `Item`: `ERROR` / `UNKNOWN` to `NaN`
- Before filling the price, `ERROR` and `UNKNOWN` in `Item` were converted to `NaN`. Otherwise they would be treated as two extra "items" and their median price (which differed from the mean) would be used to fill prices.

### Step 5. Fill `Price Per Unit`, then `Total Spent`

| Missing | Derived from |
|---|---|
| `Price Per Unit` | the item's median price, then `Total Spent ÷ Quantity` |
| `Total Spent` | `Quantity × Price Per Unit` |

- The median was used for the price because each known item has one fixed price (median equals mean), so there is no outlier risk.
- `Quantity` was filled again after `Total Spent` was completed.

**Remaining after filling:** `Quantity` 23, `Total Spent` 23, `Price Per Unit` 6 rows. These cannot be derived because two of the three values are missing in the same row.
**Decision:** leave them `NaN` (not estimated) and mark them with `Numeric_missing = True`. In total **26 rows** have at least one of the three values missing (the same row can be missing more than one). They are ignored automatically in sums and averages.

### Step 6. Infer `Item` from the price

| Price | Item |
|---|---|
| 1.0 | Cookie |
| 1.5 | Tea |
| 2.0 | Coffee |
| 3.0 | Cake **or** Juice |
| 4.0 | Sandwich **or** Smoothie |
| 5.0 | Salad |

- **Decision:** fill the item only when the price identifies it uniquely (1.0, 1.5, 2.0, 5.0). Prices 3.0 and 4.0 are ambiguous, so they are **not** guessed.
- **Remaining:** 480 rows have an unknown item, all marked with `Item_missing = True`.
- **Impact:** item-level analysis covers 9,520 of 10,000 rows (95.2%).

### Step 7. `Payment Method` and `Location`
- `ERROR`, `UNKNOWN` and empty values all mean "not known", so they were merged into one value: **`unknown`**.

| Column | ERROR | UNKNOWN | Empty | Total unknown | Share |
|---|---|---|---|---|---|
| Payment Method | 306 | 293 | 2,579 | **3,178** | 31.8% |
| Location | 358 | 338 | 3,265 | **3,961** | 39.6% |

- **Why a text value and not `NaN`:** `NaN` is silently dropped from `groupby`, pivots and charts, so a third of the rows would disappear without warning. A visible `unknown` category keeps the gap in view.
- **Why not filled with the most frequent value:** the cross-tab of Payment Method vs. Location shows almost equal counts in every cell, so there is no relationship that could be used to infer a missing value.
- **Impact:** any analysis by payment method or location covers only about 60-68% of transactions. Report this next to any such result.

### Step 8. `Transaction Date`
- `ERROR`, `UNKNOWN` and empty values were converted to `NaT` with `errors='coerce'`. All valid dates are in `YYYY-MM-DD` format and fall between 2023-01-01 and 2023-12-31.
- **Decision:** no date is estimated. The transaction ID has no relationship with the date (rank correlation ≈ 0.006), so it cannot be used to infer one.
- **Remaining:** 460 rows (4.6%), marked `Date_missing = True`.
- **Impact:** these rows are excluded from time-based analysis (monthly sales, trends) only. They stay in all other analysis (item, payment, location).

---

## Flags in the output

| Column | Meaning |
|---|---|
| `Item_missing = True` | 480 rows with an unknown item |
| `Date_missing = True` | 460 rows with an unknown date |
| `Numeric_missing = True` | 26 rows with a missing `Quantity`, `Price Per Unit` or `Total Spent` |

Each flag is a separate True/False column, so a row can carry several flags and filtering is simple (for example `df[~df['Date_missing']]`).

## Final data quality summary

| Column | Missing values | Treatment |
|---|---|---|
| Transaction ID | 0 | none needed |
| Item | 480 (4.8%) | left empty + `Item_missing` flag |
| Quantity | 23 (0.2%) | left empty + `Numeric_missing` flag |
| Price Per Unit | 6 (0.06%) | left empty + `Numeric_missing` flag |
| Total Spent | 23 (0.2%) | left empty + `Numeric_missing` flag |
| Payment Method | 3,178 (31.8%) | merged into `unknown` |
| Location | 3,961 (39.6%) | merged into `unknown` |
| Transaction Date | 460 (4.6%) | `NaT` + `Date_missing` flag |

## Notes for analysis
1. Financial totals are calculated from rows that have `Total Spent` (9,977 of 10,000).
2. Always state the coverage (number of rows used) next to any result that depends on `Item`, `Payment Method`, `Location` or `Transaction Date`.
3. Do not read `unknown` as a real category when comparing payment methods or locations.
