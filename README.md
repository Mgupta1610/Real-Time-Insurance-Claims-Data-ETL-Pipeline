# **Real-Time Insurance Data ETL Pipeline with Snowflake**

A production-style, Airflow-orchestrated ETL that ingests insurance data (Kaggle), lands it in **AWS S3**, transforms/validates it, and loads curated tables into **Snowflake** for **real-time** analytics in **Tableau**.

---

## **Project Overview**
This project creates a **real-time ETL** pipeline to process insurance data from **Kaggle** and load it into **Snowflake**. The pipeline is orchestrated with **Apache Airflow**, lands raw and transformed assets in **AWS S3**, and applies data quality checks before **COPY**/**MERGE** into Snowflake. The final modeled tables are available for **near-real-time** reporting in **Tableau** via live or scheduled connections.

**Highlights**
- Incremental ingestion with idempotent tasks
- Schema-validated transformations
- Data quality gates (row counts, null/PK checks)
- Cost-aware staging (S3) and warehouse sizing (Snowflake)
- Dashboard-ready tables and views

---

## **Architecture Diagram**
<img width="837" height="478" alt="Screenshot 2025-09-07 at 7 27 25 PM" src="https://github.com/user-attachments/assets/deb59bbc-6fea-4163-94f3-08b8744ac0d7" />
---

## Tools and Services Used
- Python: For scripting and data processing.
- Pandas: For data manipulation and analysis.
- AWS S3: As a data lake for storing raw and transformed data.
- Snowflake: For data modeling and storage.
- Apache Airflow: For orchestrating the ETL workflow.
- EC2: For hosting the Airflow environment.
- Kaggle API: For extracting data from Kaggle.
- Tableau: For data visualization.

## ETL Workflow

### Data Extraction
- Create an Airflow DAG script to extract a random number of rows from the Kaggle insurance dataset using the Kaggle API.
- Clean, normalize, and transform the data into four tables:
  - policy_data
  - customers_data
  - vehicles_data
  - claims_data

### Data Storage
- Store the transformed data in S3 buckets, organized into different folders for each type of normalized data (policy_data, customers_data, vehicles_data, claims_data).

### Data Processing in Snowflake
- Create Snowflake SQL worksheets to define database schemas.
- Create staging tables in Snowflake for each type of normalized data.
- Define Snowpipes to automate data ingestion from the S3 buckets to the staging tables.
- Create stream objects for each staging table to capture changes.
- Create final tables to merge new data from the stream objects, ensuring only distinct or new rows are inserted.

### Change Data Capture with Snowflake Streams and Tasks
- Create tasks in Snowflake to automate change data capture.
- Each task is triggered when new data is available in the stream objects to load the data into the final tables.

### Airflow DAG Tasks
- Task 1: Check if the Kaggle API is available.
- Task 2: Upload the transformed data to the S3 bucket.

## Setup Instructions

### EC2 Instance Setup
- Create an EC2 Ubuntu t2.small instance to host the Airflow environment.
- Immediately after creating the instance, change inbound rules in the EC2 instance's security group to allow TCP traffic on port 8080. This allows access to the Airflow webserver.
- SSH into the EC2 instance and install dependencies as detailed in how_to_run.docx.
- Set up Python environment and install Airflow along with necessary dependencies.
- Configuration: Insert kaggle.json and the DAG script as specified in how_to_run.docx.
- Start Airflow components and access the Airflow webserver at <EC2 public IPV4 DNS>:8080.
- Update Airflow connections and trigger the DAG as outlined in how_to_run.docx.

## Running the Project
- Once the cleaned data is stored in S3, it triggers other objects in Snowflake for further preprocessing to establish change data capture.
- Access the Airflow webserver at <EC2 public IPV4 DNS>:8080 and trigger the DAG to start the ETL process.

## Data Visualization
- Install the ODBC Snowflake driver to connect Tableau to Snowflake. Instructions for installing the driver can be found on the Snowflake documentation page.
- Set up a live data visualization connection in Tableau to the final tables in Snowflake.
- Use Tableau to analyze and visualize the final transformed data for insights and reporting.

### Dashboard
<img width="833" height="709" alt="Screenshot 2025-09-07 at 7 34 42 PM" src="https://github.com/user-attachments/assets/029af2c1-52c4-4c3d-b0a6-17b583da2458" />

