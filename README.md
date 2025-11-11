# 🏗️ Data Warehouse and Analytics Project

Welcome to the **Data Warehouse and Analytics Project** repository! 🚀
This project demonstrates a **comprehensive data warehousing and analytics solution** — from building a warehouse to generating actionable insights.

Designed as a **portfolio project**, it highlights **industry best practices** in data engineering, ETL pipelines, and analytics modeling.

---

## 📘 Project Overview

This repository follows the **Medallion Architecture** (Bronze → Silver → Gold) and is implemented using **SQL scripts**.
It demonstrates how to:

* Load raw data
* Apply progressive transformations
* Build analytics-ready tables using a **Star Schema**

**Goal:** Reproducible ETL, clear lineage, and fast analytical querying.

> Inspired by the *Databricks Medallion Architecture* framework.

---

## 🧱 Architecture — Medallion + Star Schema

The **Medallion Architecture** organizes data into layers that progressively improve **data quality and business value**:

| Layer         | Purpose                                    | Example Output          |
| :------------ | :----------------------------------------- | :---------------------- |
| 🥉 **Bronze** | Raw, uncleaned data                        | Staging tables, logs    |
| 🥈 **Silver** | Cleansed, validated, standardized data     | Canonical tables        |
| 🥇 **Gold**   | Curated, business-ready data for analytics | Fact & dimension tables |

This pattern is commonly used in **modern lakehouse** and **data warehouse** systems — and works perfectly in SQL Server or Azure SQL as a logical ETL structure.

### 🔗 Medallion + Star Schema Integration

* **Silver Layer:** Holds normalized, domain-specific tables (e.g., Customer, Product, Order).
* **Gold Layer:** Combines these into denormalized **Star Schema** tables — **Facts** and **Dimensions** — for BI tools like *Power BI* or *Tableau*.

This hybrid approach ensures:

* Clear data lineage
* Scalable analytics
* Easy integration with visualization tools

---

## ⚙️ Layer Details

### 🥉 Bronze — Raw Ingestion / Staging

**Purpose:**
Capture data exactly as received, with minimal transformations. Include metadata like `source`, `ingestion_ts`, `file_name`, and `batch_id`.

**Files:** `scripts/bronze/*`

**Example — BULK INSERT**

```sql
CREATE SCHEMA bronze;

CREATE TABLE bronze.orders_raw (
  order_id        VARCHAR(50),
  customer_id     VARCHAR(50),
  order_ts        VARCHAR(50),
  raw_json        NVARCHAR(MAX),
  load_ts         DATETIME2 DEFAULT SYSUTCDATETIME(),
  source_file     VARCHAR(255)
);

BULK INSERT bronze.orders_raw
FROM 'C:\datasets\orders.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', ROWTERMINATOR = '\n');
```

---

### 🥈 Silver — Cleansing, Canonicalization, Domain Modeling

**Purpose:**
Clean, validate, and standardize the Bronze data.
Perform type conversions, deduplication, and domain modeling (Customers, Products, Events, etc.).

**Files:** `scripts/silver/*`

**Typical Steps:**

* Parse & cast datatypes (e.g., convert `order_ts` to `DATETIME2`)
* Standardize text (uppercase, trimming)
* Remove duplicates using window functions
* Map business keys to surrogate keys

**Example — Deduplication Using Window Function**

```sql
WITH ranked AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY load_ts DESC) AS rn
  FROM bronze.orders_raw
)
INSERT INTO silver.orders (order_id, customer_id, order_dt, total)
SELECT order_id,
       customer_id,
       TRY_CAST(order_ts AS DATETIME2),
       CAST(JSON_VALUE(raw_json, '$.total') AS DECIMAL(10,2))
FROM ranked
WHERE rn = 1;
```

---

### 🥇 Gold — Curated, Analytics-Ready (Star Schema)

**Purpose:**
Produce business-ready, denormalized datasets for analytics.
The **Gold** layer contains the **Star Schema** — **Fact** and **Dimension** tables.

**Files:** `scripts/gold/*`

> Inspired by *Microsoft Learn* best practices for star schema modeling.

**Typical Outputs:**

* Aggregated daily sales
* Precomputed KPIs
* Dimensional models for BI dashboards

**Example — Load FactSales from Silver**

```sql
INSERT INTO gold.fact_sales (date_key, product_key, customer_key, quantity, revenue, batch_id)
SELECT d.date_key,
       p.product_key,
       c.customer_key,
       s.quantity,
       s.price * s.quantity,
       @batch_id
FROM silver.sales AS s
JOIN gold.dim_date     AS d ON CAST(s.order_dt AS DATE) = d.date
JOIN gold.dim_product  AS p ON s.product_sku = p.sku
JOIN gold.dim_customer AS c ON s.customer_ref = c.customer_ref;
```

---

## 📊 Data Model & Analytics (Star Schema)

The **Gold** layer forms the foundation for analytics and BI reports.

**Core Tables:**

* `DimCustomer`
* `DimProduct`
* `FactSales`

**Best Practices:**

* Use surrogate keys for dimensions.
* Store `effective_start_date` and `effective_end_date` for slowly changing dimensions (SCD).
* Index foreign keys in fact tables for performance.

**Example Analytics Queries**

**1️⃣ Total Revenue by Month**

```sql
SELECT d.YearMonth,
       SUM(f.SalesAmount) AS TotalRevenue
FROM gold.fact_sales AS f
JOIN gold.dim_date AS d
  ON f.date_key = d.date_key
GROUP BY d.YearMonth
ORDER BY d.YearMonth;
```

**2️⃣ Top N Products by Region**

```sql
SELECT TOP 10
       p.ProductName,
       l.Region,
       SUM(f.SalesAmount) AS Revenue
FROM gold.fact_sales AS f
JOIN gold.dim_product AS p ON f.product_key = p.product_key
JOIN gold.dim_location AS l ON f.location_key = l.location_key
GROUP BY p.ProductName, l.Region
ORDER BY Revenue DESC;
```

**3️⃣ Customer Retention / Cohort**

```sql
SELECT c.CustomerKey,
       MIN(d.date_key) AS FirstPurchaseKey,
       COUNT(DISTINCT d.date_key) AS PurchaseMonths
FROM gold.fact_sales AS f
JOIN gold.dim_customer AS c ON f.customer_key = c.customer_key
JOIN gold.dim_date AS d ON f.date_key = d.date_key
GROUP BY c.CustomerKey
HAVING COUNT(DISTINCT d.date_key) > 1;
```

---

## 🧩 Summary

This project demonstrates:

* **ETL automation** using layered Medallion architecture
* **Dimensional modeling** for business intelligence
* **Scalable analytics** using SQL Server

It’s a complete example of how to go from **raw data → clean → insights** in a structured, maintainable way.

---

Would you like me to make this README include a **"How to Run Locally"** and **"Folder Structure"** section next?
That will make it look like a polished professional project (ideal for your GitHub portfolio).
