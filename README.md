# Medallion Architecture Data Warehouse: End-to-End ETL & Dimensional Modeling

## 🌟 Project Overview
This project demonstrates the design and implementation of an enterprise-grade Data Warehouse utilizing Microsoft SQL Server (T-SQL). It orchestrates a complete ETL/ELT pipeline following the **Medallion Architecture** (Bronze ──> Silver ──> Gold) to ingest, clean, and transform disparate, raw operational data from corporate CRM and ERP systems into a highly optimized, reporting-ready **Star Schema**. 

<sup>*A core engineering focus of this project is ensuring high data quality, seamless referential integrity via structural Surrogate Keys, and maintaining a strict standardization rule for handling missing or corrupt values.*</sup>

---

## 🏛️ Architecture & Data Flow

### 🧱 1. Bronze Layer <sub style="font-size: 12px; color: gray;">(Staging Platform)</sub>
The Bronze layer serves as the data lake/landing zone for the warehouse. 
* **Mechanism:** It implements high-performance SQL Server `BULK INSERT` queries configured with table-level locking (`TABLOCK`) to minimize transaction logging overhead.
* **Operation:** Every execution completely wipes the landing zone (`TRUNCATE TABLE`) before streaming the raw data from local `.csv` flat files. This isolates staging workloads and keeps storage footprint minimal.
* **Datasets Ingested:** 3 CRM source files (`cust_info`, `prd_info`, `sales_details`) and 3 ERP source files (`cust_az12`, `loc_a101`, `px_cat_g1v2`).

### 🥈 2. Silver Layer <sub style="font-size: 12px; color: gray;">(Standardization & Quality Assurance)</sub>
The Silver layer is where raw staging entries are transformed into validated enterprise facts and dimensions. It cleans text anomalies, enforces matching schemas, and handles business validation logic:
* **String Standardization:** Applies `TRIM()` functions across all text attributes to eliminate trailing/leading whitespace variations native to CSV transfers.
* **Missing Value Fallback:** Implements a strict data quality rule where all structural `NULL` values, missing records, or unpopulated spaces are dynamically converted to a uniform lowercase `'n/a'` literal string. This ensures clean categorical groupings in downstream reports.
* **Timeline Versioning (SCD Type 2):** For tracking historical product changes over time, it employs SQL window functions (`LEAD() OVER (...) - 1`) to dynamically calculate product lifecycle expiration fields (`prd_start_dt` to `prd_end_dt`).
* **Defensive Math Calculations:** Financial metrics are validated for accuracy (`Quantity * Price`). It applies `NULLIF(sls_quantity, 0)` during division calculations to shield the automated pipeline from catastrophic zero-division database crashes.

### 🥇 3. Gold Layer <sub style="font-size: 12px; color: gray;">(The Dimensional Star Schema)</sub>
The Gold layer models the clean tables into an analytical Star Schema, optimizing query performance for Business Intelligence (BI) software like Power BI or Tableau.
* **Surrogate Key Generation:** It decouples fragile operational text-based keys by assigning auto-incrementing integer **Surrogate Keys** (`customer_key` and `product_key`) using `ROW_NUMBER() OVER (...)`.
* **Fact Table Mapping:** The central fact table (`gold.fact_sales`) performs analytical `LEFT JOIN` operations against the clean dimensions to swap raw business keys for their new optimized surrogate integer mappings.
* **Presentation Views:** It wraps all physical dimensional tables inside decoupled database views (`gold.vw_...`). This provides a secure, permanent layer for BI consumption, meaning structural backend tables can be altered in the future without breaking frontend dashboards.

---

## 📂 Repository Structure

```text
├── scripts/
│   ├── 01_database_setup.sql       # Initializes database, schemas (bronze, silver, gold)
│   ├── 02_load_bronze_master.sql   # Bulk Insert stored procedure for raw data ingest
│   ├── 03_load_silver_master.sql   # Data cleaning, validation, and 'n/a' fallback script
│   ├── 04_load_gold_master.sql     # Surrogate key maps, Star Schema fact/dim loader
│   └── 05_analytics_views.sql      # Presentation layer views for BI connectivity
└── README.md
