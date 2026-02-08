# Sales Forecasting using Azure Databricks

An end-to-end **Lakehouse + Machine Learning project** built on **Azure Databricks**, implementing **Unity Catalog governance**, **Medallion Architecture**, **Delta Lake**, **BI Dashboards**, and **MLflow-based demand forecasting**.

This project demonstrates how raw retail data can be securely ingested, cleaned, modeled, analyzed, and finally used to train and evaluate a machine learning model for **daily sales forecasting**.

---

## 📌 Project Highlights

- Secure data ingestion from **Azure Data Lake Storage (ADLS)**
- Centralized governance using **Unity Catalog**
- **Raw → Bronze → Silver → Gold** Medallion architecture
- Delta Lake optimizations (OPTIMIZE, Z-ORDER)
- Business dashboards using Databricks SQL
- Time-series feature engineering
- Machine learning with **Spark ML + MLflow**
- Model registry, versioning, and inference
- Business-friendly model performance evaluation

---

## 🛠️ Technology Stack

- Azure Databricks
- Apache Spark (PySpark & Spark SQL)
- Delta Lake
- Unity Catalog
- Azure Data Lake Storage Gen2 (ADLS)
- MLflow
- Random Forest Regressor
- Databricks SQL Dashboards

---

## 📂 Dataset

- Retail Superstore dataset (Kaggle)(3A Superstore)
- CSV format
- Data stored in Azure Data Lake Storage

---

## 🏗️ High-Level Architecture

ADLS (CSV files)
-> Raw Layer (External Volume)
-> Bronze Layer (Delta in Volume)
-> Silver Layer (Clean Managed Tables)
-> Gold Layer (Facts, Aggregates, Features)
-> Dashboards & ML Models

---

## 🔐 Unity Catalog & ADLS Integration

### Step 1: Azure Storage Access
- Assigned **Storage Blob Data Contributor**
- Principal: `unity-catalog-access-connector` (Managed Identity)

### Step 2: Storage Credential & External Location
- Created Storage Credential using Managed Identity
- Created External Location pointing to ADLS container
- Enabled governed access through Unity Catalog

### Step 3: Volumes
- Created External Volumes on top of ADLS paths
- All data access done via `/Volumes/...`

<img width="1468" height="715" alt="Screenshot 2026-02-09 at 1 54 07 AM" src="https://github.com/user-attachments/assets/05275512-a78e-4e92-be73-6fa5e0a17bf1" />

---

## 🟤 Raw Layer

### Purpose
- Store raw CSV files exactly as received
- No transformation
- Immutable source of truth

### Implementation
- Unity Catalog External Volume
- Spark reads CSV directly from volume

<img width="1470" height="711" alt="Screenshot 2026-02-09 at 1 52 46 AM" src="https://github.com/user-attachments/assets/70d218be-e419-4854-9d64-aa819a69ff16" />


---

## 🟫 Bronze Layer

### Purpose
- Convert CSV → Delta
- Add ingestion metadata
- Preserve original structure

### Key Design Choice
- Bronze data stored as Delta **inside Unity Catalog Volumes**
- No Unity Catalog tables at Bronze level (to avoid governance conflicts)

### Added Metadata
- `ingestion_ts`
- `source_file`

<img width="1469" height="739" alt="Screenshot 2026-02-09 at 1 54 45 AM" src="https://github.com/user-attachments/assets/eb9659b5-4f8f-4bb6-9ea6-c0333e398b1b" />

---

## ⚪ Silver Layer (Cleaned Data)

### Purpose
- Data cleansing and standardization
- Remove duplicates
- Handle nulls and invalid values
- Prepare data for analytics & ML

### Tables Created
- `silver.customers`
- `silver.branches`
- `silver.categories`
- `silver.orders`
- `silver.order_details`

### Key Transformations
- Column renaming
- Data type casting
- Date validation
- Numeric normalization

<img width="1470" height="741" alt="Screenshot 2026-02-09 at 1 55 26 AM" src="https://github.com/user-attachments/assets/9534861c-61b5-4faa-9e6f-718c7dc4e9eb" />

---

## 🟡 Gold Layer (Business & ML Ready)

### 1️⃣ Sales Fact Table

**`gold.sales_fact`**

- Central transactional table
- Joins orders, order details, customers, branches, and products
- Used by dashboards and ML features

<img width="1444" height="703" alt="Screenshot 2026-02-09 at 1 55 54 AM" src="https://github.com/user-attachments/assets/0adfa3da-cd87-4547-9c7f-63fd22efaee1" />
<img width="1464" height="783" alt="Screenshot 2026-02-09 at 2 20 20 AM" src="https://github.com/user-attachments/assets/fda346ba-da30-4265-9861-ec8506cedcc5" />
---

### 2️⃣ Branch Sales (BI Aggregation)

**`gold.branch_sales`**

Daily branch-level KPIs:
- Total Revenue
- Total Orders
- Total Units Sold
- Average Order Value

Used for BI dashboards.

<img width="1470" height="733" alt="Screenshot 2026-02-09 at 1 58 59 AM" src="https://github.com/user-attachments/assets/f130e07a-0494-434a-9adb-d1f1f3d19c59" />
<img width="1470" height="816" alt="Screenshot 2026-02-09 at 2 21 08 AM" src="https://github.com/user-attachments/assets/8bc5abba-a330-4a92-b7eb-ebb65354f45f" />


---

### 3️⃣ Sales Details (ML Feature Table)

**`gold.sales_details`**

Granularity:
- Item × Branch × Day

Target:
- `DAILY_UNITS_SOLD`

Features:
- `ROLLING_7D_UNITS`
- `ROLLING_30D_UNITS`
- `LAG_1D_UNITS`

Built using SQL window functions.

<img width="1463" height="731" alt="Screenshot 2026-02-09 at 1 57 39 AM" src="https://github.com/user-attachments/assets/06d0aba3-b3e5-4421-a47d-a5b2e079532c" />
<img width="1470" height="835" alt="Screenshot 2026-02-09 at 2 21 30 AM" src="https://github.com/user-attachments/assets/f468d5c4-46a8-4f96-ad4d-5feeea039658" />


---

## ⚡ Delta Optimization

```sql
OPTIMIZE gold.sales_fact ZORDER BY (ORDER_TS, ITEM_ID);
OPTIMIZE gold.sales_details ZORDER BY (ORDER_DATE, ITEM_ID);
```

## 📊 BI Dashboard

Built using **Databricks SQL Dashboards** to provide business-friendly insights from the Gold layer.

### Visuals Included
- **Total Revenue by Branch**
- **Total Orders by Branch**
- **Total Units Sold by Branch**
- **Average Order Value**
- **Revenue Trend Over Time**

<img width="1105" height="581" alt="Screenshot 2026-02-09 at 2 04 21 AM" src="https://github.com/user-attachments/assets/bf3b5028-1fea-439d-8560-175d96c3f76b" />

---

## 🤖 Machine Learning – Sales Forecasting

### Objective
Predict **daily units sold per item per branch** using historical sales patterns.

---

### Feature Engineering
To capture time-series behavior, the following features were engineered:

- **Rolling averages**
  - Capture short-term and long-term demand trends
- **Lag feature**
  - Captures recent sales momentum
- **Time-series aware design**
  - Prevents data leakage by using only past information

---

### Model Used
- **Algorithm**: Spark ML `RandomForestRegressor`
- **Train / Test Split**: 80% / 20%
- **Evaluation Metric**: RMSE (Root Mean Squared Error)

---

## 🧪 Experiment Tracking with MLflow

MLflow was used to track and compare multiple model runs.

### Tracked Per Run
- Hyperparameters (`numTrees`, `maxDepth`)
- RMSE metric
- Model artifacts
- Input schema
- Model signature

Multiple hyperparameter combinations were tested to identify the best performing model.

<img width="1464" height="736" alt="Screenshot 2026-02-09 at 2 06 14 AM" src="https://github.com/user-attachments/assets/ad748e70-6962-4162-bd70-1733c8326753" />


---

## 🧾 Model Registry (Unity Catalog)

The best performing model was registered in **Unity Catalog** for governance and lifecycle management.

### Registry Features
- Versioned model storage
- Centralized governance
- Ready for inference or deployment

**Registered model name**
<img width="1467" height="725" alt="Screenshot 2026-02-09 at 2 09 00 AM" src="https://github.com/user-attachments/assets/f77ae849-d200-4a76-aecc-0476694b477f" />
<img width="1469" height="730" alt="Screenshot 2026-02-09 at 2 47 37 AM" src="https://github.com/user-attachments/assets/376075e6-75b9-4e68-87a0-57396950342a" />

---

## 🔮 Model Inference & Evaluation

### Inference Steps
1. Load model from Unity Catalog registry
2. Assemble feature vector
3. Generate predictions
4. Compare actual vs predicted values
5. Calculate prediction error
6. Classify model performance
   
<img width="1466" height="818" alt="Screenshot 2026-02-09 at 2 24 13 AM" src="https://github.com/user-attachments/assets/02f60691-6c62-4fa1-bbff-f86326960f25" />

---

### Business-Friendly Evaluation Logic
To make model performance understandable to non-technical users:

- **Error ≤ 2 units** → ✅ Success
- **Error > 2 units** → ❌ Failure

Final **success and failure percentages** were computed to summarize overall model quality.
<img width="1463" height="740" alt="Screenshot 2026-02-09 at 2 09 54 AM" src="https://github.com/user-attachments/assets/a9da5147-e41d-423f-a9e8-dc5fe3d002a5" />
