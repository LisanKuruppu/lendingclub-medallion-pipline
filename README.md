# Loan Prediction — Medallion Architecture Pipeline

Project: Loan prediction pipeline using Lending Club data implemented with a Bronze → Silver → Gold medallion architecture in PySpark.

**Repository layout**
- `notebooks/` — Main notebooks implementing Bronze, Silver, and Gold layers (cleaning, analytics, and ML).
    - `01_bronze_ingestion.ipynb` — Bronze layer: raw data ingestion and metadata addition.
    - `02_silver_cleaning.ipynb` — Silver layer: data cleaning, typing, and validation.
    - `03_gold_analytics_ml.ipynb` — Gold layer: aggregated business metrics and ML model training.
- `data/` — Raw and processed data folders (contains `lendingclub/` and `medallion/` subfolders).
- `spark-env/` — local Spark / virtualenv layout used for development (contains `bin/python`).

**Overview**
This project demonstrates a reproducible data engineering pipeline using Spark:
- Bronze: ingest raw CSV(s) and add metadata (partitioned parquet).
- Silver: cleaning, typing and validation (structured parquet).
- Gold: aggregated business metrics and optional ML model training for default prediction.

Key outputs are stored under `data/medallion/gold/` (aggregations, models, predictions).

Getting started

1. Python & Spark

- Recommended: use the included runtime in `spark-env/` or ensure PySpark is installed in your environment.
- Basic dependencies (if you don't have them):

```bash
pip install pyspark findspark jupyter
```

2. Run the notebook interactively

- Start Jupyter with the desired Python interpreter (for example the one in `spark-env/bin/python`):

```bash
spark-env/bin/python -m jupyter lab
```

- Open `Project/Project.ipynb` and run cells in order. The Silver layer writes cleaned parquet to `data/medallion/silver/accepted` which the Gold layer reads.

3. Execute notebook headlessly (optional)

```bash
# execute entire notebook (increase timeout for large runs)
jupyter nbconvert --to notebook --execute Project/Project.ipynb --ExecutePreprocessor.timeout=1200
```

If you only want to run the Gold layer (faster during development):

- First ensure the Silver layer output exists: `data/medallion/silver/accepted`.
- Open the notebook and run the single Gold-layer cell (or run a trimmed script that reads `data/medallion/silver/accepted` and executes the Gold logic).

Important notes about columns

- The Silver layer standardizes column names. The Gold and ML code expect cleaned names such as `loan_amount`, `interest_rate`, `annual_income`, `debt_to_income_ratio`, etc.
- If you modified column names or used a different ingestion path, update the Gold cell to point to the correct columns or to load a different path.

Outputs

- Aggregations: `data/medallion/gold/loan_status_summary`, `income_analysis`, `loan_distribution`, `rate_analysis` (each a parquet folder).
- Models & predictions (if ML is enabled): `data/medallion/gold/models/` and `data/medallion/gold/predictions/`.

Deliverables mapping (course project)

- Part 1 (Data Ingestion): Bronze layer code & raw files (found under `data/lendingclub/`).
- Part 2 (Data Cleaning): Silver layer implemented in `Project.ipynb` (PySpark map/mapPartitions + DataFrame operations). Use the notebook cells under "Silver Layer".
- Part 3 (Data Serving): Gold layer provides aggregations and a simple ML pipeline (optional) — results saved to `data/medallion/gold/` for serving to dashboards or apps.

AI Tools disclosure

- AI assistance used: GitHub Copilot (GPT-5 mini) for code suggestions and refactoring of the Gold-layer notebook cell. Purpose: refactor/standardize Gold-layer code, add aggregation saves, and provide an ML pipeline skeleton. All code changes were reviewed and adapted by the project authors.

How to extend

- Add feature engineering steps in the Silver layer and persist intermediate features.
- Replace the sample logistic regression with cross-validated models using `ml.tuning.CrossValidator` and `ParamGridBuilder` from Spark MLlib.
- Export Gold-layer aggregations to a small REST API or a dashboard tool (Streamlit, Superset, or a simple Flask app).

Contact

If you have questions, open an issue in this repo or contact the project maintainers listed in the course group submission.

---
Generated README for the medallion pipeline project.
