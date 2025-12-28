
# 🎬 Netflix Analytics Engineering Project (dbt + Snowflake)

## 📌 Project Overview
This project demonstrates an **end-to-end Analytics Engineering workflow** using **dbt** and **Snowflake**, built on top of the **MovieLens dataset** to simulate a real-world **Netflix-like analytics platform**.

The objective of this project is to showcase how raw data can be transformed into **analytics-ready fact and dimension tables** using modern data engineering best practices.

---

## 🧠 Business Context
Netflix-style platforms generate massive volumes of data such as:
- User ratings and engagement events
- Movie metadata and genres
- User-generated tags and feedback
- External movie references (IMDb, TMDB)

Analytics teams rely on clean, reliable, and well-modeled datasets to power dashboards, experimentation, and downstream machine learning use cases.

---

## 🏗️ Architecture Overview

MovieLens CSV Files  
→ Amazon S3 (Raw Storage)  
→ Snowflake RAW Schema  
→ dbt Staging Models (SRC)  
→ dbt Dimension & Fact Models (DEV)  
→ dbt Docs (Auto-generated Documentation)

---

## 📂 Dataset Used
**MovieLens 20M Dataset**

Files:
- `ratings.csv` – User ratings with timestamps  
- `movies.csv` – Movie titles and genres  
- `tags.csv` – User-generated tags  
- `links.csv` – External references (IMDb, TMDB)  
- `genome-tags.csv` – Tag definitions  
- `genome-scores.csv` – Tag relevance scores per movie  

---

## ⚙️ Tools & Technologies
- dbt Core
- Snowflake
- Amazon S3
- SQL
- YAML
- GitHub

---

## 🧩 Project Structure

```
netflixdbt/
│
├── models/
│   ├── sources/
│   ├── staging/
│   ├── dimensions/
│   └── facts/
│
├── screenshots/
│   ├── 01_project_structure.png
│   ├── 02_dbt_run_success.png
│   ├── 03_snowflake_raw_tables.png
│   ├── 04_snowflake_dev_schema.png
│   ├── 05_dbt_docs_lineage.png
│   └── 06_dbt_docs_model.png
│
├── dbt_project.yml
├── packages.yml
└── README.md
```

---

## 🧱 Data Modeling Layers

### 1️⃣ Raw Layer
- CSV files loaded into Snowflake without modification
- Acts as a permanent source of truth

### 2️⃣ Staging Layer (`src_*`)
- Column renaming
- Data type casting
- Basic data quality cleanup

### 3️⃣ Dimension Tables
- `dim_movies`
- `dim_users`
- `dim_genome_tags`

### 4️⃣ Fact Tables
- `fct_ratings`
- `fct_genome_scores`

---

## 🧠 Key dbt Models & SQL Logic

Below are selected SQL snippets from this project that demonstrate how dbt models were built
using staging layers, reusable references, and dimensional modeling best practices.

### 🔹 Staging Model – Movies (`src_movies.sql`)

```sql
select
    movie_id,
    title,
    genres
from {{ source('netflix', 'movies') }}
```

---

### 🔹 Dimension Model – Movies (`dim_movies.sql`)

```sql
{{ config(materialized='table') }}

select
    movie_id,
    title,
    genres
from {{ ref('src_movies') }}
```

---

### 🔹 Fact Model – Ratings (`fct_ratings.sql`)

```sql
{{ config(materialized='incremental') }}

select
    user_id,
    movie_id,
    rating,
    rating_timestamp
from {{ ref('src_ratings') }}

{% if is_incremental() %}
where rating_timestamp >
    (select max(rating_timestamp) from {{ this }})
{% endif %}
```

---

### 🔹 Dimension Model – Users (`dim_users.sql`)

```sql
with ratings_users as (
    select distinct user_id from {{ ref('src_ratings') }}
),
tag_users as (
    select distinct user_id from {{ ref('src_tags') }}
)

select distinct user_id
from ratings_users
union
select distinct user_id
from tag_users
```

---

## 📸 Screenshots

### 🔹 Project Folder Structure
![Project Structure](screenshots/01_project_structure.png)

### 🔹 dbt Run – Successful Execution
![dbt Run](screenshots/02_dbt_run_success.png)

### 🔹 Snowflake RAW Tables
![Snowflake Raw](screenshots/03_snowflake_raw_tables.png)

### 🔹 Snowflake DEV Schema
![Snowflake Dev](screenshots/04_snowflake_dev_schema.png)

### 🔹 dbt Docs – Lineage Graph
![dbt Lineage](screenshots/05_dbt_docs_lineage.png)

---

## 📘 dbt Documentation
```bash
dbt docs generate
dbt docs serve
```

---

## 🚀 How to Run the Project

1. Upload MovieLens CSV files to Amazon S3  
2. Load raw data into Snowflake  
3. Initialize the dbt project  
4. Configure Snowflake profile  
5. Run transformations:
```bash
dbt run
```

---

## 👤 Author
**Kushal Jain**  
MS in Information Systems (Business Analytics)  
Analytics Engineering | Data Engineering | BI
