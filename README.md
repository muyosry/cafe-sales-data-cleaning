# Cafe Sales Data Cleaning

Cleaning a messy 10,000-row café sales dataset with Python and pandas, using a **no-guessing** approach: a value is filled only when it can be derived from the same row. Everything else is kept as missing and marked with a flag column.

**Source data:** [Dirty Cafe Sales on Kaggle](https://www.kaggle.com/datasets/omniam0hamed/dirty-cafe-sales)

## Summary

| | Raw data | Cleaned data |
|---|---|---|
| Rows | 10,000 | 10,000 (no rows dropped) |
| Placeholder values (`ERROR`, `UNKNOWN`, blanks) | Mixed into every column | Converted to proper missing values or the `unknown` category |
| Numeric columns | Stored as text | `float64` |
| `Transaction Date` | Text | `datetime64` |
| `Total Spent` = `Quantity` × `Price Per Unit` | Not verifiable | 0 mismatches |

## Key decisions

- **Never guess.** `Total Spent`, `Quantity` and `Price Per Unit` are recovered only from the other two, because `Total Spent = Quantity × Price Per Unit`.
- **Item is inferred from price only when the price is unambiguous.** Cookie (1.0), Tea (1.5), Coffee (2.0) and Salad (5.0) are unique prices. Prices 3.0 (Cake/Juice) and 4.0 (Sandwich/Smoothie) are shared, so those items stay missing.
- **Text columns → `unknown`.** `Payment Method` and `Location` have no reliable way to be recovered (the crosstab shows near-equal counts), so missing values become `unknown`. This keeps them visible in groupby and pivot tables, which silently drop `NaN`.
- **Dates → `NaT` + flag.** The Transaction ID has no relationship to the date (rank correlation ≈ 0.006), so dates cannot be reconstructed.

Full reasoning for every step is in [`cleaning_decisions.md`](cleaning_decisions.md).

## Flag columns

| Flag | Meaning | True rows |
|---|---|---|
| `Item_missing` | Item could not be determined | 480 |
| `Date_missing` | Transaction date is missing or invalid | 460 |
| `Numeric_missing` | At least one of Quantity / Price Per Unit / Total Spent is missing | 26 |

## Remaining missing values (intentional)

| Column | Missing |
|---|---|
| `Item` | 480 |
| `Quantity` | 23 |
| `Price Per Unit` | 6 |
| `Total Spent` | 23 |
| `Transaction Date` | 460 |
| `Payment Method` = `unknown` | 3,178 |
| `Location` = `unknown` | 3,961 |

**Use with care:** time-based analysis excludes the 460 rows without a date, and item-level analysis excludes the 480 rows without an item. Use the flags to filter them in or out explicitly.

## Repository contents

| File | Description |
|---|---|
| `dirty_cafe_sales.csv` | Raw dataset (original, uncleaned) |
| `dirty_cafe_sales.ipynb` | The cleaning notebook, step by step with explanations |
| `output.csv` | The cleaned dataset (11 columns, including the 3 flags) |
| `cleaning_decisions.md` | Detailed documentation of each cleaning decision |

## How to run

The raw file `dirty_cafe_sales.csv` is included in this repository (original source: [Kaggle](https://www.kaggle.com/datasets/omniam0hamed/dirty-cafe-sales)).

1. Clone or download this repository.
2. Install the requirements:
   ```bash
   pip install pandas numpy jupyter
   ```
3. Open `dirty_cafe_sales.ipynb` and run all cells. The cleaned data is written to `output.csv`.

## Tools

Python · pandas · NumPy · Jupyter Notebook

## License

The dataset is released under [CC0 (Public Domain)](https://creativecommons.org/publicdomain/zero/1.0/) by its original author. The code and notebook are available under the MIT license.
