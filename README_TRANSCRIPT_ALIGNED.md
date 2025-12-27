# Netflix-Style Analytics Engineering Project (dbt + Snowflake)

## Project Overview
This repository contains an **end-to-end analytics engineering project** built using **dbt** on **Snowflake**, following the exact concepts and workflow explained in the accompanying project transcript/tutorial.

The project simulates a **Netflix-style analytics environment** using the MovieLens dataset and focuses on how real analytics engineering teams:
- transform raw warehouse data into analytics-ready datasets
- enforce data quality through testing
- manage performance using incremental models
- track historical changes using snapshots (SCD Type 2)
- document data models and lineage for analysts and stakeholders

This is not a simple SQL project — it demonstrates **production-grade analytics modeling practices**.

---

## What problem this project solves
In real organizations, raw data:
- arrives from multiple sources
- is messy and inconsistently formatted
- cannot be queried directly for reporting or analytics

Analytics engineers solve this by creating a **semantic / analytics layer** that:
- standardizes data
- applies business logic once
- guarantees data quality
- is easy for analysts and BI tools to consume

This project demonstrates exactly that workflow using dbt.

---

## High-level architecture
```
Raw MovieLens Data (CSV)
        ↓
Snowflake RAW schema
        ↓
dbt transformations
  - staging models
  - dimension models
  - fact models
  - marts
        ↓
Analytics-ready tables
        ↓
BI / reporting / analysis
```

The ingestion of CSV files into Snowflake is assumed to be done prior to running dbt.  
This repository focuses on **warehouse modeling and analytics engineering**.

---

## Repository structure
```
analyses/            # Ad-hoc analysis queries (not materialized)
macros/              # Reusable Jinja/SQL macros
models/
  staging/           # Source-cleaning models (close to raw)
  dim/               # Dimension tables
  fct/               # Fact tables
  mart/              # Business-ready analytics marts
  schema.yml         # Documentation and data tests
seeds/               # Reference data loaded via dbt seed
snapshots/           # SCD Type 2 snapshot definitions
tests/               # Custom SQL-based tests
```

---

## Data modeling layers (explained simply)

### 1. Staging models (`models/staging/`)
**Purpose:** Create clean, consistent representations of raw data.
- Minimal transformations
- Standardized column names
- Type and formatting cleanup

Examples:
- `src_movies`
- `src_ratings`
- `src_tags`
- `src_genome_tags`
- `src_genome_score`
- `src_links`

Staging models act as the **contract** between raw data and business logic.

---

### 2. Dimension models (`models/dim/`)
**Purpose:** Describe business entities used for analysis.

Dimensions in this project:
- **dim_movies**
  - Cleans movie titles
  - Parses genre strings into arrays
- **dim_users**
  - Builds a distinct user list from ratings and tags
- **dim_genomes_tags**
  - Standardizes tag labels for readability
- **dim_movies_with_tags** (ephemeral)
  - Combines movies, tags, and relevance scores
  - Not persisted as a table; compiled into downstream SQL

---

### 3. Fact models (`models/fct/`)
**Purpose:** Store measurable events and metrics.

Facts in this project:
- **fct_ratings** (incremental)
  - Stores user ratings by movie
  - Uses timestamp-based incremental logic to load only new records
- **fct_genome_scores**
  - Stores relevance scores between movies and genome tags
  - Applies rounding and filtering logic

Incremental modeling demonstrates how dbt scales to large datasets.

---

### 4. Mart models (`models/mart/`)
**Purpose:** Provide business-ready datasets optimized for reporting.

- **mart_movie_releases**
  - Combines curated dimensions and facts
  - Enriched with seeded reference data
  - Designed for consumption by dashboards or analysts

---

## dbt features demonstrated
This project intentionally demonstrates core dbt capabilities discussed in the transcript:

- **Materializations**
  - view
  - table
  - incremental
  - ephemeral
- **Testing**
  - not_null
  - unique
  - relationships
  - custom SQL tests
- **Snapshots**
  - SCD Type 2 history tracking
- **Seeds**
  - Reference data managed in version control
- **Documentation**
  - Column-level descriptions
  - Auto-generated lineage graphs

---

## Data quality and testing
Data quality is enforced using dbt tests defined in `schema.yml` and custom SQL tests.

Tests ensure:
- primary keys are unique and not null
- foreign key relationships are valid
- business rules (such as relevance score constraints) are met

Run tests using:
```bash
dbt test
```

---

## Snapshots (Slowly Changing Dimensions)
Snapshots track changes in mutable data over time while preserving history.

This project includes:
- **snap_tags**
  - Tracks changes in tag assignments using a composite key
  - Captures historical versions automatically

Run snapshots using:
```bash
dbt snapshot
```

---

## Seeds (reference data)
Seeds are small CSV files committed to the repository and loaded into Snowflake.

They are used to:
- enrich mart models
- provide consistent lookup/reference data

Load seeds using:
```bash
dbt seed
```

---

## How to run the project

### Prerequisites
- Snowflake account with raw MovieLens data loaded
- dbt installed locally
- Valid dbt profile configured (do not commit `profiles.yml`)

### Commands
```bash
dbt deps
dbt run
dbt test
dbt docs generate
dbt docs serve
```

The documentation site opens locally (usually at `http://localhost:8080`).

---

## How to review this project (for recruiters)
If you have limited time:
1. Review the `models/` folder to understand the layered design
2. Open `schema.yml` to see documentation and tests
3. Check `fct_ratings.sql` for incremental logic
4. Check `snapshots/snap_tags.sql` for SCD Type 2 implementation
5. View dbt Docs to explore lineage and column metadata

---

## Security and best practices
- Credentials (`profiles.yml`) are not included
- Generated artifacts (`target/`, `logs/`, `dbt_packages/`) are excluded
- Version control is used only for source-controlled assets

---

## Future enhancements
- CI/CD with GitHub Actions for dbt validation
- Public hosting of dbt Docs via GitHub Pages
- Additional marts for KPI and cohort analysis
- BI dashboard integration

---

## Author
**Kushal Jain**  
MS in Information Systems (Business Analytics) – Florida International University  
Expected Graduation: December 2025  
LinkedIn: https://www.linkedin.com/in/kushaljaink/
