# 📦 Clickcartel – End-to-End Spark Data Engineering Pipeline

## 🚀 Project Overview

You are a Data Engineer at **Clickcartel**, a rapidly growing e-commerce platform.  
The marketing and product teams struggle to make data-driven decisions because:

- User event data is messy  
- Customer & product data is disconnected  
- No unified analytics layer exists  

They approach you with a critical business need:

> **“We need to understand our user journey.  
What products are people viewing?  
What actions lead to purchases?  
Who are our most valuable customers?”**

Your mission:

### ✔️ Build an automated, scalable **multi-layered Spark/Delta Lake pipeline**  
✔️ Process raw user event streams  
✔️ Clean & enrich data with customer/product information  
✔️ Produce aggregated **Gold** tables for analytics  

This pipeline becomes Clickcartel’s **single source of truth** for user behavior.

---

## 📊 Data Sources (Synthetic Generation)

### **1️⃣ Raw User Events (JSON, Streaming)**
- Ingested via **Structured Streaming (Auto Loader)**
- Contains:
  - `timestamp`
  - `user_id`
  - `event_type` → `view_product`, `add_to_cart`, `purchase`
  - `product_id`
- **Intentionally skewed** so certain products receive disproportionately high number of views.

### **2️⃣ Customer Profiles (CSV, Batch)**
- Dimension table with:
  - `customer_id`
  - `signup_date`
  - `location`

### **3️⃣ Product Details (Parquet, Batch)**
- Product catalog with:
  - `product_id`
  - `product_name`
  - `category`
  - `price`

---

# 🧱 Section 1: Apache Spark Architecture & Components

This project demonstrates Spark fundamentals:

- **Execution hierarchy** → jobs, stages, tasks  
- **Lazy evaluation** → transformations build the DAG before actions  
- Modules used:
  - Spark SQL  
  - DataFrames / Dataset API  
  - Structured Streaming  

---

# 🧠 Section 2: Spark SQL Concepts Used

### ✔️ Reading multiple formats
- JSON (Auto Loader)  
- CSV  
- Parquet  

### ✔️ Writing Delta tables
- `append` for streaming  
- `overwrite` for batch refresh  

### ✔️ Partitioning
- Gold tables use `partitionBy()` for optimized query performance.

---

# 🛠️ Section 3: Spark DataFrame / Dataset API Techniques

### 🔹 Column Manipulation
- `withColumn()`  
- `withColumnRenamed()`  
- `select()`  

### 🔹 Deduplication
- Stream-to-stream deduplication using:
  ```python

  withWatermark("event_time", "10 minutes").dropDuplicates(["event_id"])
## ⚙️ Section 4: Troubleshooting & Performance Tuning

### 🎯 Handling Data Skew
- The raw user events dataset is intentionally skewed — a few products receive disproportionately high numbers of views.
- To mitigate this, the Gold layer implements a **salting technique**:
  - Adds a random salt key before aggregation
  - Distributes skewed keys across partitions
  - Reduces shuffle pressure and eliminates long-tail tasks

### 🎯 Partitioning Strategy
- Gold tables are written using `partitionBy()` for faster downstream queries.
- Common partition columns include:
  - `event_date`
  - `product_category`
- Optimizes BI tools like Power BI or Tableau.

### 🎯 Adaptive Query Execution (AQE)
- AQE is enabled by default on Databricks.
- It provides automatic optimization for:
  - **Skew join handling**
  - **Dynamic shuffle partitioning**
  - **Optimized join strategies**
- Greatly improves performance for skewed datasets like Clickcartel's event logs.

---

## 🔄 Section 5: Structured Streaming

### ✔️ End-to-End Streaming Pipeline
- The pipeline from **Bronze → Silver** is implemented using **Structured Streaming**.
- Auto Loader (`cloudFiles`) ingesting raw JSON ensures:
  - Incremental file discovery
  - Schema inference
  - Schema evolution handling
