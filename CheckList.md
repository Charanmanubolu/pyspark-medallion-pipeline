### 1. The Utilities (`src/utils/`)

**File: `spark_session.py`**
- [ ] **Define `get_spark_session(app_name="MedallionPipeline")`:** Write a `try/except` block. Try to return the global `spark` object (for when it runs in Databricks). In the `except NameError` block, build and return a local `SparkSession` (for local testing).

**File: `helpers.py`**
- [ ] **Define `add_audit_columns(df)`:** Take a DataFrame as input. Add an `_inserted_ts` column using PySpark's `current_timestamp()`. Return the DataFrame.
- [ ] **Define `generate_surrogate_key(df, key_col_name, source_cols)`:** Take a DataFrame, the new column name (e.g., `"customer_sk"`), and a list of columns to hash. Use `concat_ws` and `md5` to generate the hash. Return the DataFrame.

---

### 2. The Bronze Layer (`src/bronze/`)

**File: `ingest_raw.py`**
- [ ] **Define `ingest_day(spark, day_folder)`:**
- [ ] Read the raw Customers JSON, Products JSON, and Orders CSV from the specified `day_folder` parameter.
- [ ] Call your `add_audit_columns()` helper function on all three DataFrames.
- [ ] Write the three DataFrames to your `data/output/bronze/` directory in Parquet format using `.mode("append")`.

---

### 3. The Silver Layer (`src/silver/`)

**File: `dim_products.py` (SCD Type 1)**
- [ ] **Define `process_products(spark)`:** Read the complete Bronze products table.
- [ ] **Clean:** Standardize the `category` column (fix typos like "Elect" to "Electronics"). Cast the `price` column to Double.
- [ ] **Deduplicate (SCD1):** Group/Window by `prod_id` and order by `event_ts` descending. Keep *only* the most recent row (row number 1). 
- [ ] Write the clean, unique records to the Silver products table using `.mode("overwrite")`.

**File: `dim_customers.py` (SCD Type 2)**
- [ ] **Define `process_customers(spark)`:** Read the Bronze customers table.
- [ ] **Clean:** Capitalize the `name` column using the native PySpark `initcap()` function. 
- [ ] **Track History (SCD2):** Compare incoming Bronze data against the existing Silver table. 
  - [ ] If a customer ID exists but the city or tier changed, mark the old record's `is_active = False` and set its `end_date`.
  - [ ] Insert the new record with `is_active = True` and a null `end_date`.
- [ ] Generate a `customer_sk` surrogate key using your helper function.
- [ ] Write to the Silver customers table.

**File: `fact_orders.py`**
- [ ] **Define `process_orders(spark)`:** Read the Bronze orders table.
- [ ] **Clean Dates:** Use the `coalesce(try_to_date(...))` trick to standardize `order_date` into a proper DateType.
- [ ] **Clean Amounts:** Use `regexp_replace` to strip letters, then cast to Double.
- [ ] **Join Dimensions:** Join clean orders to Silver `dim_products` and `dim_customers` to retrieve their Surrogate Keys (`product_sk`, `customer_sk`). 
- [ ] Drop the old raw string IDs (`C-100`, `P-01`).
- [ ] Write to the Silver orders table using `.mode("append")`.

---

### 4. The Gold Layer (`src/gold/`)

**File: `business_metrics.py`**
- [ ] **Define `calculate_running_totals(spark)`:** Read Silver `fact_orders`. Partition by `customer_sk`, order chronologically by `order_date`, and sum the amount. Write this aggregated view to Gold.
- [ ] **Define `calculate_tier_revenue(spark)`:** Join Silver `fact_orders` with Silver `dim_customers`. Filter for `is_active = True`. Group by the customer `tier` and sum the revenue. Write this summary table to Gold.

---

### 5. The Orchestrator

**File: `main.py`**
- [ ] **Define `run_pipeline(day_folder)`:**
- [ ] Import all processing functions from the `src/` directories.
- [ ] Initialize Spark using your `get_spark_session()` utility.
- [ ] Execute sequentially: Bronze Ingest -> Silver Dimensions -> Silver Facts -> Gold Metrics.
- [ ] Create an `if __name__ == "__main__":` block.
- [ ] Call `run_pipeline("raw_day1")`, print a success message.
- [ ] Call `run_pipeline("raw_day2")` to execute and verify the incremental CDC logic.


