Data Warehouse and Analytics Project


Welcome to the Data Warehouse and Analytics Project repository! 🚀
This project demonstrates a comprehensive data warehousing and analytics solution, from building a data warehouse to generating actionable insights. Designed as a portfolio project, it highlights industry best practices in data engineering and analytics.


Project overview

This repository demonstrates a data warehousing project that organizes data using the Medallion architecture (Bronze → Silver → Gold), implemented with T-SQL scripts that load raw data, apply progressive transformations, and produce analytics-ready tables (star schema). The goal: reproducible ETL, clear lineage, and fast analytics. 
Databricks

Architecture — Medallion + Star Schema

Medallion architecture organizes data into layers that increase in quality and readiness: Bronze (raw), Silver (cleaned & integrated), Gold (curated, business-ready). This pattern is commonly used in lakehouse and modern data platforms but is equally useful as a logical structure for SQL-based ETL. 

How medallion + star schema fit together: model normalized or enterprise domain tables in Silver; create denormalized, consumption-optimized star-schema tables in Gold (facts + dimensions). This hybrid gives strong lineage and performant reporting. 


Layer details:

Bronze — Raw ingestion / staging
Purpose: land data exactly (or nearly exactly) as received, plus basic metadata (source, ingestion_ts, file_name, batch_id). Keep it immutable for lineage.
Files: scripts/bronze/*
Example BULK INSERT


Silver — Cleansing, canonicalization, domain modeling

Purpose: apply validation, typing, deduplication, normalization, and create canonical domain tables (customers, products, events). This is the place for joins across sources and SCD handling.
Files: scripts/silver/*
Typical steps:
Cast/parse datetimes, standardize text (e.g., UPPER/LOWER, trimming).
Remove duplicate event rows (use window functions).
Map source keys to enterprise keys; create surrogate keys if required.
Example dedupe with window function


Gold — Curated, analytics-ready (star schema)

Purpose: produce business-ready denormalized datasets (facts + dimensions). Design tables to support reporting and BI tools (Power BI, Tableau). Typical patterns: aggregated daily facts, precomputed KPIs, and star-schema tables for fast queries. 
Microsoft Learn
Files: scripts/gold/*
Example: load FactSales from Silver


Data model & Analytics (Star schema)

Build DimDate, DimCustomer, DimProduct, DimLocation, and FactSales in the Gold layer; keep surrogate keys and load effective dates if implementing SCD types. The Gold layer often holds the star schema used by reports. 
Sample analytics queries (run against gold.fact_* and gold.dim_*):
Total revenue by month
Top N products by region
Customer cohort retention using first purchase date



