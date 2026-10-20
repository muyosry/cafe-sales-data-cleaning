# ☕ Cafe Sales Data Cleaning & Processing Pipeline

![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

An end-to-end Python data cleaning and feature engineering project designed to handle messy, incomplete, and corrupted sales data from a retail/cafe environment. 

This project transforms raw transactional data (`dirty_cafe_sales.csv`) containing missing values, mathematical inconsistencies, erroneous entries, and unstandardized text into a production-ready clean dataset (`output.csv`).

---

## 📌 Project Overview

In real-world data analytics, datasets rarely come clean. This project addresses common data quality issues found in transactional datasets, including:

* **Missing Numerical Values:** Imputing missing prices, quantities, and totals using domain-specific logic and descriptive statistics.
* **Text Normalization:** Standardizing category names, payment methods, and store locations.
* **Date Parsing & Imputation:** Formatting mixed-string dates and handling missing timestamps.
* **Audit Flags Creation:** Adding metadata flags (`Item_missing`, `Date_missing`, `Numeric_missing`) to keep track of imputed records for auditability.

---

## 🛠️ Data Cleaning Pipeline Workflow

```
[Raw Data: dirty_cafe_sales.csv] (10,000 Records)
                     │
                     ▼
         1. Numerical Imputation & Math Recalculation
   (Quantity * Price Per Unit = Total Spent)
                     │
                     ▼
         2. Categorical Normalization & Handling Unknowns
   (Standardize Location, Payment Methods, Items)
                     │
                     ▼
         3. Date Parsing & Formatting
   (Unified YYYY-MM-DD Format)
                     │
                     ▼
         4. Feature Engineering / Audit Flag Creation
   (Track imputed values without losing original context)
                     │
                     ▼
[Clean Data: output.csv] (Production Ready)
```

---

## 🔍 Key Data Cleaning Logic

### 1. Mathematical Consistency & Recalculation
* **If `Total Spent` is missing:** Calculated via `Quantity` × `Price Per Unit`.
* **If `Price Per Unit` is missing:** Inferred using `Total Spent` / `Quantity` or filled using the item's `Median` price.
* **If `Quantity` is missing:** Derived via `Total Spent` / `Price Per Unit`.

### 2. Categorical Cleaning
* Unstandardized text values like `UNKNOWN`, `ERROR`, or blank strings in `Payment Method` and `Location` are grouped into standardized categories.

### 3. Auditability (Missing Value Flags)
To maintain transparency, binary indicator columns were created:
* `Item_missing`: `1` if the item name was reconstructed/missing, `0` otherwise.
* `Date_missing`: `1` if the transaction timestamp was imputed, `0` otherwise.
* `Numeric_missing`: `1` if any numerical field required calculation/imputation, `0` otherwise.

---

## 📁 Repository Structure

```text
.
├── dirty_cafe_sales.csv   # Original dirty dataset
├── output.csv             # Cleaned and processed dataset
├── data_cleaning.py       # Main Python processing script
├── README.md              # Project documentation
└── requirements.txt       # Dependencies
```

---

## 🚀 How to Run

### 1. Clone the Repository
```bash
git clone https://github.com/muyosry/cafe-sales-data-cleaning.git
cd cafe-sales-data-cleaning
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Run the Cleaning Script
```bash
python data_cleaning.py
```

---

## 📊 Results Summary

| Metric | Pre-Cleaning | Post-Cleaning |
| :--- | :--- | :--- |
| **Total Rows** | 10,000 | 10,000 |
| **Missing Prices/Totals** | Present | Fully Resolved (0 Missing) |
| **Date Inconsistencies** | Mixed Formats | Standardized `YYYY-MM-DD` |
| **Categorical Errors** | Present (`ERROR`, `UNKNOWN`) | Standardized Categories |
| **Data Readiness** | Unusable for ML/BI | Ready for Analysis & Visualization |

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](../../issues).

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.