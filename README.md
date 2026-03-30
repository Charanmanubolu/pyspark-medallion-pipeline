# PySpark Medallion Pipeline: E-Commerce Logistics Lakehouse

## 📌 Project Overview
This repository contains a production-grade Data Engineering pipeline built with PySpark. It implements the **Medallion Architecture** (Bronze, Silver, Gold) to ingest, clean, and transform messy, real-world e-commerce logistics data into a highly optimized Star Schema.

The pipeline is designed to simulate a multi-day batch load, specifically showcasing advanced Data Warehousing concepts like **Slowly Changing Dimensions (SCD Type 1 & 2)**, surrogate key generation, and chronological window functions.

## 🏗️ Architecture & Data Flow

1. **🥉 Bronze Layer (Raw Data):** Ingests messy JSON and CSV files. Standardizes column names and appends ingestion audit timestamps (`_inserted_ts`).
2. **🥈 Silver Layer (Cleaned & Modeled):**
   * **`dim_products` (SCD Type 1):** Cleans category typos, casts prices to Double, and overwrites old records with new updates.
   * **`dim_customers` (SCD Type 2):** Tracks customer history (e.g., tier upgrades, city moves) using `start_date`, `end_date`, and `is_active` flags.
   * **`fact_orders`:** Cleans messy date strings (`yyyy-MM-dd` vs `MM/dd/yyyy`), drops bad amounts, and joins to Dimension tables using hashed Surrogate Keys.
3. **🥇 Gold Layer (Business Metrics):**
   * **Running Totals:** Calculates the cumulative spend per customer.
   * **Aggregations:** Calculates total revenue by city and active customer tier.

## 🛠️ Tech Stack
* **Language:** Python 3.x
* **Engine:** Apache Spark (PySpark)
* **Storage Format:** Parquet / Delta (Simulated Local Data Lake)

## 📂 Repository Structure
```text
pyspark-medallion-pipeline/

├── README.md                 
├── requirements.txt          
├── main.py       
├── .gitignore      
├── src/                      
│   ├── bronze/               
│   │   └── ingest_raw.py        
│   ├── silver/               
│   │   ├── dim_customers.py     
│   │   ├── dim_products.py      
│   │   └── fact_orders.py 
│   ├── gold/                 
│   │   └── business_metrics.py  
│   └── utils/                
│       ├── spark_session.py     
│       └── helpers.py           
└── data/                     
    ├── raw_day1/             
    ├── raw_day2/             
    └── output/