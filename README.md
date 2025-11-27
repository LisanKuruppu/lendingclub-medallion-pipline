# Loan Prediction — Medallion Architecture Pipeline

**Repository:** [https://github.com/LisanKuruppu/lendingclub-medallion-pipline](https://github.com/LisanKuruppu/lendingclub-medallion-pipline)

**Project:** Distributed loan default prediction pipeline using Lending Club data (2007–2018), implemented with a Bronze → Silver → Gold Medallion Architecture on a **3-node Apache Spark Cluster**.

## 📂 Repository Layout

- **`notebooks/`** — Core pipeline logic divided by architectural layer.
    - **`01_bronze_ingestion.ipynb`** — **Ingestion:** Raw CSV loading using RDDs, schema validation, and conversion to Parquet.
    - **`02_silver_cleaning.ipynb`** — **Cleaning:** Low-level RDD MapReduce transformations, string sanitization, deduplication, and outlier filtering.
    - **`03_gold_analytics.ipynb`** — **Serving:** Spark SQL business analytics, feature engineering, and MLlib model training (Random Forest/Logistic Regression).
- **`data/`** — Data storage (Shared via NFS across the cluster).
    - `lendingclub/` — Raw CSV source files.
    - `medallion/` — Processed data (`bronze/`, `silver/`, `gold/`).
- **`spark-env/`** — Python virtual environment configuration used by Driver and Executors.

## 🏗️ Infrastructure & Architecture

This project is designed to run on a **distributed Apache Spark Standalone Cluster**, not just a local machine.

### Cluster Topology
- **Master Node:** Spark Master, Jupyter Server, NFS Server.
- **Worker Nodes (x2):** Spark Workers (4 Cores, 8GB RAM each), NFS Clients.
- **Networking:** Custom DNS resolution via `/etc/hosts` and SSH communication.
- **Storage:** **NFS (Network File System)** implemented to resolve "Split-Brain" storage issues, ensuring all workers see a unified view of the data.

### Medallion Layers
1.  **Bronze:** Raw data ingestion. Preserves history with metadata columns (`_ingestion_timestamp`, `_source_file`).
2.  **Silver:** Data quality enforcement using **Spark MapReduce** (no SQL/DataFrames used for logic).
    - *Key Features:* Custom Python cleaning functions, Null/Whitespace sanitization, Deduplication via `reduceByKey`.
3.  **Gold:** Business-level aggregates and Machine Learning.
    - *Analytics:* Risk segmentation by FICO score, geographic analysis, and loan grade trends.
    - *ML:* Prediction of Loan Default (`loan_status_binary`) using Spark MLlib Pipelines.

## 🚀 Getting Started

### 1. Prerequisites
- Python 3.11+
- Java 8 (OpenJDK)
- Apache Spark 3.5.0
- NFS (if running distributed) or Local File System (if running standalone)

### 2. Python Environment
Install dependencies on all nodes (Master and Workers):
```bash
pip install pyspark findspark jupyter numpy pandas
