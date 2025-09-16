# Airflow-CICD

This repository implements a **CI/CD pipeline** for orchestrating **flight booking data processing** using **Apache Airflow (Composer)**, **Google Cloud Dataproc Serverless**, and **BigQuery**.  

The pipeline automates:
- Uploading environment variables to Cloud Composer.
- Deploying Airflow DAGs to Composer environments.
- Deploying PySpark transformation jobs to GCS.
- Running data transformations and loading results into BigQuery.

---

## 📂 Repository Structure

