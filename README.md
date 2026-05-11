# Spare Parts Inventory Analytics
### End-to-End ETL Pipeline & Dashboard | PySpark · Python · Excel

---

## The Problem

Factory spare parts data is messy by nature — brand names entered inconsistently, location formats varying across warehouses, unit price columns typed as strings instead of numbers. Without clean data, there is no reliable way to answer basic operational questions:

- Which parts are most critical to stock?
- Which warehouse is overloaded?
- Where is money tied up in low-turnover inventory?

This project takes raw, unstructured spare parts data and builds a complete analytics pipeline from scratch — cleaning, structuring, and surfacing insights that support real inventory decisions.

---

## What This Project Does

```
Raw CSV  →  Schema Enforcement  →  Data Profiling  →  Cleaning  →  Feature Engineering  →  Analytics  →  Dashboard
```

**Dataset:** 12,000+ rows of factory spare parts records across multiple warehouse areas, brands, and part categories.

---

## Data Quality Issues Found & Fixed

This is the core of the project — not just loading data, but finding what was wrong with it before touching any analysis.

| Issue | What Was Found | Fix Applied |
|---|---|---|
| Type mismatch | `Unit Price` loaded as StringType by inferSchema | Defined schema manually using `StructType` with `FloatType` |
| Case inconsistency | Same brand entered as `Siemens` and `SIEMENS` | Detected via lowercase groupBy + `collect_set`; standardised |
| Whitespace pollution | Leading/trailing spaces in Brand Name, Part Name, Location | Detected using `trim()` comparison; cleaned across columns |
| Location separator mismatch | Mix of `-` and `–` (en-dash) in warehouse location field | Detected using `contains()` checks; flagged and standardised |
| Invalid values | Checked for Unit Price ≤ 0 and Quantity ≤ 0 | Validated and filtered |
| Duplicate rows | Checked exact duplicates and key-combination duplicates | Identified using `groupBy().count().filter(count > 1)` |

> **Why this matters:** Every one of these issues would have broken a `groupBy` or `JOIN` silently — producing wrong numbers without any error. Catching them before analysis is what separates production-grade data work from notebook experiments.

---

## Feature Engineering

Three new columns derived from raw data to enable meaningful analysis:

```python
# Total monetary value of each part in stock
total_stock_value = unit_price × quantity

# Warehouse area extracted from full location string
warehouse_area = split(location, " - ")[0]

# Part category mapped from part name using domain knowledge
# PLC / IO / HMI  → Automation
# Motor / VFD     → Drives
# Bearing / Belt  → Mechanical
# Sensor          → Sensors
# Cable           → Electrical
# Oil / Coolant   → Consumables
part_category = mapped using when().otherwise() logic

# Risk score: high value + low quantity = procurement risk
risk_score = unit_price / quantity
```

---

## Analytics Performed

**Inventory Analytics**
- Top 5 most expensive parts by unit price
- Top 10 parts by total stock value
- Total inventory value per warehouse area
- Total quantity per part category
- Average unit price per brand
- Brand contributing highest inventory value
- Warehouse area with maximum stock quantity

**Risk & Operational Analysis**
- Parts with quantity < 5 (low stock alert)
- High value + low quantity items (unit_price > ₹20,000 and qty < 3)
- Parts stored in only one warehouse (single point of failure risk)
- Risk score ranking — top 5 most critical parts to reorder

**Ranking & Prioritisation**
- Parts ranked within each brand by unit price using `dense_rank()` Window Function
- Warehouse areas ranked by total inventory value using `rank()` Window Function

---

## Excel Dashboard

An Excel dashboard was built on top of the cleaned and aggregated data, covering:

- **KPI Summary** — Total inventory value, total part variants, high-risk item count, low-stock alerts
- **Top 10 Parts by Stock Value** — horizontal bar chart
- **Inventory Value by Warehouse Area** — column chart
- **Category Distribution** — pie chart showing Sensors / Automation / Drives / Mechanical / Electrical / Consumables
- **Average Unit Price by Brand** — brand-level pricing comparison
- **High Risk Parts Table** — top 5 parts ranked by risk score with part details

---

## Project Structure

```
spare-parts-inventory-analytics/
│
├── spare_parts_analysis.ipynb     # Full PySpark ETL + analytics notebook
├── SpareParts_Dashboard.xlsx      # Excel dashboard with charts and KPI cards
├── spare_parts_clean.csv          # Cleaned and enriched output dataset
└── README.md
```

---

## Tools & Techniques

| Area | Tools Used |
|---|---|
| Data Processing | PySpark (SparkSession, DataFrame API) |
| Schema Control | StructType, StructField — manual schema definition |
| Data Cleaning | isNull, isnan, trim, lower, collect_set, contains |
| Feature Engineering | withColumn, when/otherwise, split |
| Analytics | groupBy, agg, sum, avg, countDistinct, orderBy |
| Window Functions | Window.partitionBy, dense_rank, rank |
| Visualisation | Excel (Charts, Conditional Formatting, KPI Cards) |
| Environment | Kaggle Notebooks |

---

## Key Findings

- **Servo motors and servo drives** carry the highest risk scores — high unit cost combined with quantities of 1–2 units
- **Area B** holds the highest inventory value, driven by drive and motor stock
- **Yaskawa and ABB** have the highest average unit prices across their part range
- **Sensors and Automation** parts account for the largest share of total part variants
- **4 parts** were flagged as low-stock (quantity < 5) requiring priority reorder

---

## Why I Built This

I work on a manufacturing shopfloor where spare parts data lives in Excel sheets and MES systems — and decisions about what to stock, where to store it, and when to reorder are often made on gut feeling rather than data.

This project was built to prove that the same structured, data-driven approach used in software analytics can be applied directly to factory operations data — with real impact on procurement, warehouse planning, and operational risk.

---

## Author

**A. Karthikeya** — Junior Automation Engineer, Foxconn Interconnect Technologies

[LinkedIn](https://www.linkedin.com/in/addepalli-karthikeya) · [Kaggle Notebook](https://www.kaggle.com/code/addepallikarthikeya/spare-parts-management-analysis)
