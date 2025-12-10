# 📘 Metadata-Driven Incremental API Ingestion Using Azure Data Factory

## 📌 Project Overview
This project implements a **metadata-driven, incremental ingestion pipeline** using **Azure Data Factory (ADF)** to fetch data from **NewsAPI** via HTTP.  
The pipeline supports incremental extraction, pagination, watermarking, and checkpointing using a **control table**, and loads the data into **Azure Data Lake Storage (ADLS)** in a clean, production-ready structure.

This project reflects real-world data engineering design patterns used in enterprise systems.

---

## 🚀 Key Features
- Metadata-driven ingestion  
- Control table–based state management  
- Incremental load using watermarks (`lastRun`, `currentMax`)  
- Pagination handling using **ADF Until loop**  
- Dynamic HTTP GET calls to NewsAPI  
- Raw data ingestion into ADLS with date partitioning  
- Automatic checkpoint update after each run  
- Git-integrated ADF project with branching  
- ARM template generation via `adf_publish` for CI/CD

---

## 🏗 Architecture Flow

### **1. Read Watermark From Control Table**
The pipeline begins by reading a SQL control table containing:

| Column        | Description |
|---------------|-------------|
| `lastRun`     | Last successful ingestion timestamp |
| `currentMax`  | Maximum timestamp extracted during the current run |
| `hasMore`     | Indicates if more API pages exist |
| `page`        | Current page number |

---

### **2. Initialize ADF Variables**
Control table values are stored in ADF variables:

- `lastRun`  
- `currentMax`  
- `hasMore`  
- `page`  

These values dynamically drive the pipeline flow.

---

### **3. UNTIL Loop for Pagination & Incremental Fetch**
An **Until** activity loops until:

```
@equals(variables('hasMore'), false)
```

Inside the loop:

#### ✔ HTTP GET (Web Activity)
Makes a dynamic API call with:

```
?page=@{variables('page')}
&from=@{variables('lastRun')}
```

#### ✔ Copy Activity → ADLS
API responses are written to:

```
raw/newsapi/YYYY/MM/DD/run_timestamp.json
```

#### ✔ Update Variables
- Extract latest `publishedAt` timestamp → update `currentMax`  
- Determine if more pages are available → update `hasMore`  
- Increment the `page` counter  

This ensures the pipeline continues until all available pages are processed.

---

### **4. Update Control Table**
Once pagination ends:

- `lastRun` = `currentMax`  
- Reset `page` = 1  
- Reset `hasMore` = false  

This prepares the pipeline for the next incremental run.

---

## 🔧 Technologies Used
- Azure Data Factory (ADF)  
- Azure SQL Database (Control Table)  
- Azure Data Lake Storage Gen2  
- NewsAPI (HTTP Source)  
- Lookup Activity  
- Set Variable Activity  
- Web Activity  
- Copy Activity  
- Until Loop  
- GitHub for version control  
- ARM templates via `adf_publish`

---

## 🗂 Repository Structure

```
/pipeline            → ADF pipeline JSON definitions
/dataset             → API, SQL, and ADLS datasets
/linkedService       → Connections to APIs, SQL DB, ADLS
/factory             → Factory-level configuration
/adf_publish         → ARM templates for CI/CD deployments
README.md
```

---
