End-to-End ETL Pipeline:
Medallion Architecture to Star SchemaAn end-to-end Data Engineering project implementing a Medallion Architecture (Bronze $\rightarrow$ Silver $\rightarrow$ Gold) using SQL Server / T-SQL. This project automates the ingestion of raw, disparate operational source data (CRM and ERP systems), cleanses and standardizes it, and models it into an optimized Star Schema (Dimensional Model) ready for BI reporting.📌 Project Overview & ArchitectureThis repository contains the complete SQL-based ETL pipeline that transforms raw transactional data into an enterprise data warehouse.The data moves through three distinct layers:Bronze (Raw/Staging): Truncates staging tables and utilizes high-performance BULK INSERT to ingest raw CSV flat files directly from source folders.Silver (Cleaned/Standardized): Cleanses data by handling null values, deduplicating records, correcting invalid entries (e.g., standardized gender codes), and enforcing uniform data types.Gold (Analytical/Dimensional): Houses the final analytical layer modeled into a Star Schema with fully optimized Dimension and Fact tables.
Tech Stack & Concepts
Database Engine: Microsoft SQL Server (T-SQL)

Data Modeling: Dimensional Modeling (Star Schema, Fact & Dimension Tables)

ETL Framework: Medallion Architecture (Bronze, Silver, Gold)

Automation & Orchestration: Stored Procedures, Windows Batch Scripts / SQL Server Agent

📐 Data Model (Star Schema)
The analytical Gold layer is structured as a Star Schema to maximize query performance for business intelligence tools like Power BI or Tableau.

Fact Table: * gold.fact_sales: Centralized transaction data containing metrics like sales amount, quantity, and foreign keys to all dimensions.

Dimension Tables:

gold.dim_customers: Unified customer profile merging CRM data with ERP location/demographic attributes.

gold.dim_products: Standardized product catalog details.

gold.dim_date: A robust calendar dimension supporting time-intelligence queries.
Step-by-Step Implementation
1. Bronze Layer (Ingestion)
The bronze.load_bronze stored procedure handles high-speed data loading. It safely truncates old staging data before running a BULK INSERT with table locking (TABLOCK) enabled to minimize transaction log overhead.

Features: Complete error tracking using BEGIN TRY / CATCH blocks and execution time logging.

2. Silver Layer (Data Cleansing)
Data from different source systems is often messy. The Silver layer transformation applies the following rules:

Standardizes codes (e.g., mapping miscellaneous inputs to uniform 'Male', 'Female', or 'Unspecified' values).

Casts generic string fields into proper DATE, INT, or DECIMAL data types.

Flags and handles missing data or orphans using COALESCE logic.

3. Gold Layer (Dimensional Transformation)
Builds the final business model. This step builds out the dimensions and links them to the primary facts using consistent, uniform keys.
Key Takeaways & Performance Optimization
Performance: Utilizing TRUNCATE instead of DELETE significantly reduced execution times and transaction log bloat during heavy staging loads.

Data Quality: Implementing strict COALESCE rules and cross-table checks successfully prevented data gaps between the CRM and ERP source tables.

Scannability: Complete PRINT logging built directly into the procedures makes tracking pipeline progress straightforward and transparent.
