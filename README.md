# Healthcare-project

## Project Goal

The goal of this project is to design and implement a scalable, secure, and efficient end-to-end data pipeline for analyzing synthetic Medicare claims data using Azure and Snowflake. By leveraging CMS’s 2008–2010 DE-SynPUF (Data Entrepreneurs Synthetic Public Use File), the project aims to simulate a real-world healthcare data engineering workflow involving:

- Ingesting multi-year healthcare datasets from on-prem-like sources into Azure Data Lake using Azure Data Factory and Self-hosted Integration Runtime (SHIR).
- Transforming and normalizing raw claims and beneficiary data into a Snowflake data warehouse using SQL-based logic with CTEs, window functions, and UDFs.
- Validating data quality with embedded Python scripts to ensure schema integrity and reduce downstream data issues.
- Enabling secure, policy-compliant handling of sensitive credentials using Azure Key Vault.
- Building analytics-ready data models to support reporting use cases for clinical, financial, and operational stakeholders.
- Delivering insights through interactive Power BI dashboards focused on utilization, chronic condition prevalence, drug costs, and overall healthcare spend trends.

## What is in the Data

This project uses the **CMS 2008–2010 Data Entrepreneurs Synthetic Public Use File (DE-SynPUF) Sample 1**, which contains synthetic Medicare claims and beneficiary data designed for research and training purposes. The dataset includes:

### Beneficiary Summary Files (2008, 2009, 2010)
- Demographic information (age, sex, race)
- Chronic condition flags (diabetes, CHF, cancer, etc.)
- Insurance coverage duration
- Date of birth and death (if applicable)

### Inpatient Claims (2008–2010)
- Hospital admission and discharge details
- ICD-9 diagnosis and procedure codes
- DRG codes
- Medicare payment amounts and patient deductibles

### Outpatient Claims (2008–2010)
- Services provided at clinics, ERs, or outpatient centers
- HCPCS/CPT codes
- Provider and claim-level reimbursement data

### Prescription Drug Events (PDE) (2008–2010)
- National Drug Codes (NDC)
- Drug cost details (total, patient-paid, plan-paid)
- Prescriber and pharmacy identifiers

### Carrier Claims Part B (2008–2010)
- Physician/supplier services and durable medical equipment
- ICD-9 diagnoses and HCPCS procedure codes
- Reimbursement, deductible, and coinsurance amounts

> Each claim is linked to a synthetic beneficiary through `DESYNPUF_ID`, enabling multi-layered analysis and patient-level joining across all claim types.

## 🛠️ Tools Used & Purpose

This project utilizes a cloud-native, enterprise-grade tool stack to simulate a production-style healthcare data pipeline. Below is a list of tools and the purpose each one serves:

| Tool / Platform              | Purpose                                                                 |
|-----------------------------|-------------------------------------------------------------------------|
| **Azure Data Factory (ADF)** | Orchestration and automation of data ingestion from on-prem to cloud   |
| **Self-hosted Integration Runtime (SHIR)** | Securely connects local CSV files to Azure services, simulating on-prem systems |
| **Azure Data Lake Storage Gen2** | Scalable and secure storage for raw and intermediate datasets          |
| **Snowflake**                | Cloud data warehouse used for scalable transformations, joins, and analytics |
| **Python**                   | Used for data validation, schema checks, and enforcing data quality rules |
| **Azure Key Vault**          | Stores secrets and credentials securely to ensure compliance and protection |
| **Power BI**                 | Data visualization and interactive dashboarding for end users          |
| **Azure DevOps + Git**       | Version control, CI/CD, and deployment automation for SQL and pipeline scripts |
| **Jira & Confluence**        | Agile sprint tracking, project documentation, and team collaboration   |




## Data Transformations

The CMS DE-SynPUF dataset includes multiple healthcare-related files that require cleaning, enrichment, and normalization before they are analytics-ready. Below are the key transformations performed throughout the ETL pipeline:

---

### 1. Data Cleaning
- Remove duplicate rows based on unique claim IDs or beneficiary IDs
- Drop or impute records with null or invalid dates (e.g., admission/discharge dates)
- Standardize column data types (e.g., ensure numeric fields are not read as strings)
- Normalize date formats (`YYYY-MM-DD`)

---

### 2. Data Normalization & Structuring
- Split wide diagnosis and procedure code columns (ICD-9) into separate normalized tables
- Explode multiple codes per claim into row-level structures for filtering and joins
- Flatten nested relationships between claims and diagnosis codes

---

### 3. Feature Engineering
- **Age calculation** from date of birth and service year
- **Age grouping** (e.g., 65–70, 71–75, etc.)
- **Chronic condition grouping** using binary flags from beneficiary summary
- **Total claim cost calculation** combining Medicare paid, patient deductible, and third-party payments
- Create flags for high-cost patients and frequent utilizers

---

### 4. Data Integration
- Join **beneficiary summary** with **inpatient, outpatient, PDE, and carrier claims** using `DESYNPUF_ID`
- Link **prescription events** with drug cost and prescriber details
- Join diagnosis codes with condition groupings for clinical analysis

---

### 5. Temporal & Window Transformations
- Rank claims by date or cost using **window functions**
- Calculate **rolling averages** (e.g., monthly spend per patient)
- Derive **readmission metrics** based on discharge → next admission date

---

### 6. Aggregations & Summarizations
- Aggregate claims data:
  - Total cost per patient per year
  - Cost per DRG or diagnosis category
  - Count of admissions per beneficiary
- Aggregate drug events by:
  - Top 10 medications by cost
  - Total prescriptions by prescriber

---

### 7. Data Validation Rules (via Python or SQL)
- Schema validation: ensure expected column names and types
- Null checks: identify and log missing critical fields
- Referential integrity: verify joins (e.g., all claims have valid beneficiary IDs)
- Range checks for dates, amounts, and codes

---

### 8. Analytics-Ready Output
- Create final fact and dimension tables:
  - `FACT_CLAIMS`, `FACT_PRESCRIPTIONS`
  - `DIM_BENEFICIARY`, `DIM_DIAGNOSIS`, `DIM_PROVIDER`
- Build **materialized views** for Power BI dashboards
- Apply **partitioning and clustering** on large tables for performance (e.g., by year, state, or condition)


## 👥 Stakeholders & Reporting Objectives

This project serves multiple healthcare and operational teams by transforming raw Medicare claims data into analytics-ready formats and actionable dashboards. Below are the primary stakeholder groups and the reporting insights tailored to each.

---

### 🩺 1. Clinical & Care Management Team
**Who:** Physicians, nurses, care coordinators, case managers  
**Reports & Insights:**
- Chronic condition prevalence (e.g., diabetes, COPD, CHF)
- Readmission rates by condition, age, and facility
- Length of stay metrics and DRG distribution
- High-risk patient identification (multiple comorbidities, frequent visits)

> 🎯 *Support clinical decision-making, improve patient care planning, and reduce avoidable admissions.*

---

### 💰 2. Finance & Reimbursement Team
**Who:** Financial analysts, CFOs, billing departments  
**Reports & Insights:**
- Total and average claim costs per beneficiary and per DRG
- Medicare vs. patient payment contributions
- Trends in prescription drug costs
- Provider-level reimbursement summaries

> 🎯 *Track cost efficiency, detect outliers, and optimize reimbursement strategies.*

---

### 🛡️ 3. Insurance & Payer Relations Team
**Who:** Payers, claims processing teams, policy analysts  
**Reports & Insights:**
- Claims volume and approval/denial ratios
- High-cost claims and patients over time
- Cost breakdowns by Part A, B, and D coverage
- Yearly utilization vs. payment trend analysis

> 🎯 *Improve payer negotiations, manage claim patterns, and ensure fair cost distribution.*

---

### 📊 4. Executive Leadership
**Who:** CXOs, VPs, board members, strategic leadership  
**Reports & Insights:**
- KPI dashboards (e.g., average claim cost, chronic condition burden)
- Cost and utilization trends across years and geographies
- Top facilities and services driving healthcare spend
- Compliance and policy impact metrics

> 🎯 *Guide high-level planning, investments, and policy decisions.*

---

### 🧪 5. Data Governance & Compliance Team
**Who:** Data stewards, audit officers, compliance leads  
**Reports & Insights:**
- Data quality and validation logs
- Schema conformance and referential integrity checks
- Missing or invalid claim counts
- Access tracking and security compliance logs

> 🎯 *Ensure data accuracy, regulatory compliance (HIPAA), and audit-readiness.*

---
