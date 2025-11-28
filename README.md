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
    

## 🏗️ Infrastructure & Architecture

This project is designed to run on a **distributed Apache Spark Standalone Cluster** (1 Master, 2 Workers).

### Medallion Layers
1.  **Bronze:** Raw data ingestion. Preserves history with metadata columns (`_ingestion_timestamp`, `_source_file`).
2.  **Silver:** Data quality enforcement using **Spark MapReduce** (no SQL/DataFrames used for logic).
    - *Key Features:* Custom Python cleaning functions, Null/Whitespace sanitization, Deduplication via `reduceByKey`.
3.  **Gold:** Business-level aggregates and Machine Learning.
    - *Analytics:* Risk segmentation by FICO score, geographic analysis, and loan grade trends.
    - *ML:* Prediction of Loan Default (`loan_status_binary`) using Spark MLlib Pipelines.

## 🛠️ Cluster Setup Guide

Since this project runs on a self-managed cluster, the following configurations are required on the Virtual Machines (VMs).

### 1. Network Configuration (`/etc/hosts`)
*Apply this to **ALL** nodes (Master and Workers) so they can resolve each other.*

```bash
# Edit /etc/hosts
192.168.18.110  spark-master
192.168.18.145  spark-worker1
192.168.18.186  spark-worker2
```

### 2. Environment Setup
*Run on **ALL** nodes to install dependencies and create the Python Virtual Environment.*

```bash
# Install System Deps
sudo apt update && sudo apt install -y openjdk-8-jdk nfs-common python3.11 python3.11-venv

# Create Virtual Env (Crucial for numpy serialization)
python3.11 -m venv ~/spark-env
source ~/spark-env/bin/activate
pip install pyspark findspark jupyter numpy pandas
```

### 3. NFS Shared Storage (Split-Brain Solution)
*To ensure all workers see the same data files, we set up NFS.*

**On Master Node (Server):**
```bash
sudo apt install nfs-kernel-server
# Create export directory
mkdir -p /home/ubuntu/lendingclub-medallion-pipline
# Configure export
echo "/home/ubuntu/lendingclub-medallion-pipline *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

**On Worker Nodes (Clients):**
```bash
mkdir -p /home/ubuntu/lendingclub-medallion-pipline
sudo mount spark-master:/home/ubuntu/lendingclub-medallion-pipline /home/ubuntu/lendingclub-medallion-pipline
```

### 4. Start Scripts

**Start Master (Run on Master):**
```bash
export SPARK_HOME=/opt/spark
export PYSPARK_PYTHON=/home/ubuntu/spark-env/bin/python3 # Force venv python

# Start Master
$SPARK_HOME/sbin/start-master.sh

# Start Jupyter
source ~/spark-env/bin/activate
nohup jupyter lab --no-browser --ip=0.0.0.0 --port=8888 --allow-root &
```

**Start Worker (Run on All Nodes):**
```bash
export SPARK_HOME=/opt/spark
export PYSPARK_PYTHON=/home/ubuntu/spark-env/bin/python3 # Force venv python

# Connect to Master
$SPARK_HOME/sbin/start-worker.sh spark://spark-master:7077 -c 4 -m 4g
```

## 🚀 Running the Pipeline

1.  **Access Jupyter:** Open `http://<MASTER_PUBLIC_IP>:8888`.
2.  **Run Notebooks in order:**
    1.  `01_bronze_ingestion.ipynb`
    2.  `02_silver_cleaning.ipynb`
    3.  `03_gold_analytics.ipynb`

## 📊 Outputs (Gold Layer)
Located in `data/medallion/gold/`:
- **Analytics:** `default_rate_by_grade`, `loan_analysis_by_state`, `risk_segments_by_fico`.
- **ML:** `models/default_prediction_model`, `risk_scores/`.

## 🎓 Deliverables Mapping

| Part | Description | Implementation |
| :--- | :--- | :--- |
| **Part 1** | Data Ingestion | `01_bronze_ingestion.ipynb`: RDD textFile loading and Parquet storage. |
| **Part 2** | Data Cleaning | `02_silver_cleaning.ipynb`: **Strict RDD/MapReduce implementation** for cleaning logic, filtering, and profiling. |
| **Part 3** | Data Serving | `03_gold_analytics.ipynb`: High-level Spark SQL analytics and MLlib pipelines. |

## 🤖 AI Tools Disclosure
**Tools Used:** ChatGPT (OpenAI)
**Scope:** Debugging Spark stack traces (`Py4JJavaError`, `ExecutorLostFailure`), generating NFS configuration syntax, and optimizing cluster resource allocation (`spark.executor.memory`).