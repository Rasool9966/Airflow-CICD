# Airflow-CICD

This repository implements a **CI/CD pipeline** for orchestrating **flight booking data processing** using **Apache Airflow (Composer)**, **Google Cloud Dataproc Serverless**, and **BigQuery**.  

The pipeline automates:
- Uploading environment variables to Cloud Composer.
- Deploying Airflow DAGs to Composer environments.
- Deploying PySpark transformation jobs to GCS.
- Running data transformations and loading results into BigQuery.

---

## 📂 Repository Structure

├── .github/workflows/ci-cd.yml # GitHub Actions workflow (CI/CD pipeline)
├── src/
│ ├── airflow_job/airflow_job.py # Airflow DAG definition
│ └── spark_job/spark_job.py # PySpark transformation script
├── variables/
│ ├── dev/variables.json # Airflow variables for DEV environment
│ └── prod/variables.json # Airflow variables for PROD environment

---

## ⚙️ CI/CD Workflow

The GitHub Actions workflow (`ci-cd.yml`) is triggered on pushes to `dev` and `main` branches:

### 🔹 Dev Deployment (`dev` branch)
1. **Upload Variables JSON** → to Composer dev bucket.  
2. **Import Variables into Airflow-DEV** → using `gcloud composer environments run`.  
3. **Upload Spark Job** → to GCS path:  
   `gs://airflows-projects-buckets/flight-booking-analysis/spark-job/`.  
4. **Upload Airflow DAG** → to Composer DEV DAG bucket.  

### 🔹 Prod Deployment (`main` branch)
1. **Upload Variables JSON** → to Composer prod bucket.  
2. **Import Variables into Airflow-PROD**.  
3. **Upload Spark Job** → to GCS.  
4. **Upload Airflow DAG** → to Composer PROD DAG bucket.  

---

## 🚀 Airflow DAG

**File:** [`airflow_job.py`](src/airflow_job/airflow_job.py)  

The DAG:
- Waits for a **source CSV file** in GCS (`flight_booking.csv`).  
- Submits a **PySpark job** to **Dataproc Serverless**.  
- Transforms the data and loads results into **BigQuery**.  

DAG ID: `flight_booking_dataproc_bq_dag`

---

## 🔥 PySpark Job

**File:** [`spark_job.py`](src/spark_job/spark_job.py)  

Steps performed:
1. Reads raw flight booking CSV from GCS.  
2. Performs transformations:  
   - Weekend flag  
   - Lead time categorization  
   - Booking success rate calculation  
3. Aggregates insights:  
   - Route-level insights  
   - Booking origin insights  
4. Writes results into BigQuery tables (overwrite mode).  

Arguments:
- `--env` → environment (`dev`, `prod`)  
- `--bq_project` → BigQuery project ID  
- `--bq_dataset` → BigQuery dataset name  
- `--transformed_table` → table for transformed data  
- `--route_insights_table` → table for route insights  
- `--origin_insights_table` → table for origin insights  

---

## 🔑 Requirements

- **Google Cloud Composer** (Airflow 2.x)  
- **Dataproc Serverless** (PySpark)  
- **BigQuery** enabled  
- GitHub secrets:
  - `GCP_SA_KEY` → Service account JSON key  
  - `GCP_PROJECT_ID` → GCP Project ID  

---

## ▶️ Running Locally

You can test the PySpark job locally (with Spark installed):

```bash
spark-submit src/spark_job/spark_job.py \
  --env dev \
  --bq_project my-gcp-project \
  --bq_dataset flight_data_dev \
  --transformed_table transformed_bookings \
  --route_insights_table route_insights \
  --origin_insights_table origin_insights
