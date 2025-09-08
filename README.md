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

## **ETL Workflow**

### **Data Extraction**
- Create an **Airflow DAG** that:
  - Pulls a *random number of rows* from the Kaggle insurance dataset via the **Kaggle API**
  - Cleans/normalizes/transforms into four normalized tables:
    - `policy_data`
    - `customers_data`
    - `vehicles_data`
    - `claims_data`

### **Data Storage (S3)**
- Write transformed outputs to **S3** with folder layout by entity:
  - `s3://<bucket>/normalized/policy_data/…`
  - `s3://<bucket>/normalized/customers_data/…`
  - `s3://<bucket>/normalized/vehicles_data/…`
  - `s3://<bucket>/normalized/claims_data/…`

### **Data Processing in Snowflake**
1. Create Snowflake **database/schemas** (e.g., `STAGE`, `CORE`, `MART`) in SQL worksheets  
2. Create **staging tables** for each normalized entity  
3. Define **Snowpipes** to auto-ingest from the S3 folders into staging tables  
4. Create **Streams** on each staging table to capture changes  
5. Create **final (CORE)** tables and **MERGE** only distinct/new rows from the streams

### **Change Data Capture (Streams & Tasks)**
- Define **Tasks** to run when stream changes are available, loading deltas into final tables automatically

### **Airflow DAG Tasks (example)**
- **Task 1:** Check Kaggle API availability  
- **Task 2:** Upload transformed artifacts to the target S3 bucket  

---

## **Setup Instructions**

### **EC2 Instance Setup**
1. Create an **Ubuntu `t2.small`** EC2 instance to host Airflow  
2. Update the **security group** inbound rules to allow **TCP :8080** (Airflow Webserver)  
3. **SSH** into the instance and install dependencies (see `how_to_run.docx`)  
4. Create Python env and install **Airflow** + providers  
5. Configure:
   - Place `kaggle.json` in the expected path (per `how_to_run.docx`)
   - Add your **DAG** file(s) under Airflow’s `dags/`
6. Start Airflow components and open:  
   `http://<EC2_PUBLIC_IPV4_DNS>:8080`
7. Configure **Airflow Connections** (AWS/Snowflake) and **trigger** the DAG (per `how_to_run.docx`)

---

## **Running the Project**
- On DAG trigger, cleaned data lands in **S3**, which in turn activates **Snowpipe/Streams/Tasks** for CDC and loading into final tables  
- Access Airflow UI at:  
  `http://<EC2_PUBLIC_IPV4_DNS>:8080`  
- Manually **Trigger DAG** to validate end-to-end run

---

## **Data Visualization (Tableau)**
1. Install **Snowflake ODBC driver** (see Snowflake docs for your OS)  
2. In **Tableau**, connect to **Snowflake** (account/warehouse/db/schema)  
3. Use the **final tables** (e.g., in `MART`) for modeling and visuals  
4. Configure **Live** or **Scheduled Extract** refresh for near-real-time insights

### **Dashboard**
- Build dashboards on top of final curated tables (examples):
  - **Claims Overview:** volumes, paid amounts, loss ratios  
  - **Customer Segments:** tenure, premium bands, churn risk  
  - **Operations:** pipeline latency, file freshness, failed loads
