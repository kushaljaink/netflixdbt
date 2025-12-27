# Netflix Analytics Engineering Project (dbt + Snowflake)

## What this repository is
This is an **end-to-end analytics engineering project** built with **dbt** on **Snowflake**, using the **MovieLens** dataset to simulate a Netflix-like analytics environment.

The goal is to show how raw data can be transformed into a **trusted analytics layer** using modern best practices:
- layered modeling (staging → dimensions/facts → marts)
- reproducible SQL transformations
- automated **data quality tests**
- **incremental** loading for large fact tables
- **snapshots** (SCD Type 2) for history tracking
- clear **documentation and lineage** via dbt docs

If you’re reviewing this as a recruiter/hiring manager: this repo demonstrates the “serve” layer of a real data platform—where analysts and BI tools consume curated tables.

---

## Business context (Netflix-style analytics)
A streaming platform generates events like:
- users rating movies (ratings fact)
- users applying tags (tags events)
- metadata about movies (movies dimension)
- relevance scores linking movies to tags (genome scores fact)

In real teams, this data is typically loaded into a warehouse in a **raw** form first. dbt then creates curated, analytics-ready tables to support use cases like:
- content performance analysis by genre
- user engagement analysis
- tag-based discovery and recommendation signals
- reporting dashboards and KPI monitoring

---

## Architecture (high level)
```
Raw MovieLens files (CSV)
        ↓
Snowflake RAW schema (loaded by ingestion step)
        ↓
dbt transformations + tests + snapshots
        ↓
DEV / ANALYTICS schema outputs (dims, facts, marts)
        ↓
BI / reporting / analysis
```

> Note: This repo focuses on the **dbt + warehouse modeling** portion. The ingestion step (loading CSVs to Snowflake) is assumed to be done before running dbt models.

---

## Repository structure
```
analyses/            # Ad-hoc analysis queries (not materialized as models)
macros/              # Reusable SQL/Jinja macros (if any)
models/
  staging/           # Source-cleaned models (light transformations; close to raw)
  dim/               # Dimension tables (entities like movies, users, tags)
  fct/               # Fact tables (events/transactions like ratings, scores)
  mart/              # Business-ready tables designed for reporting
  schema.yml         # Model + column descriptions and tests
seeds/               # Small reference datasets committed to repo (CSV)
snapshots/           # Snapshot definitions (SCD Type 2 history tracking)
tests/               # Custom tests (SQL assertions)
```

---

## Data layers explained (simple)
### 1) Staging (`models/staging/`)
**Purpose:** standardize raw fields and create a reliable starting point.
- Minimal business logic
- Naming consistency
- Type/format cleanup

Typical outputs: `src_movies`, `src_ratings`, `src_tags`, `src_genome_tags`, `src_genome_score`, `src_links`

### 2) Dimensions (`models/dim/`)
**Purpose:** create descriptive entities used to slice facts.
- `dim_movies`: clean movie titles, parse genres into an array
- `dim_users`: unique user set derived from ratings and tags
- `dim_genomes_tags`: cleaned tag labels used in genome scoring

### 3) Facts (`models/fct/`)
**Purpose:** store measurable events/metrics.
- `fct_ratings` (**incremental**): user ratings by movie with timestamp
- `fct_genome_scores`: relevance scores between movie and genome tags

### 4) Marts (`models/mart/`)
**Purpose:** business-ready tables optimized for reporting/BI.
- `mart_movie_releases`: enriches movie-related reporting with release-date availability and other derived flags

---

## Models in this project (what each one does)
### Staging models
- **src_movies**: raw movie metadata (movie_id, title, genres)
- **src_ratings**: raw user ratings (user_id, movie_id, rating, rating_timestamp)
- **src_tags**: raw user tags (user_id, movie_id, tag, tag_timestamp)
- **src_genome_tags**: tag dictionary (tag_id, tag)
- **src_genome_score**: relevance scores (movie_id, tag_id, relevance)
- **src_links**: external IDs (imdb_id, tmdb_id)

### Dimension models
- **dim_movies**
  - cleans title formatting (trim/initcap)
  - converts genre string (`Action|Comedy`) into an array for analytics
- **dim_users**
  - builds a distinct user list from ratings + tags
- **dim_genomes_tags**
  - standardizes tag text into a clean `tag_name`
- **dim_movies_with_tags** (ephemeral)
  - joins movies + tags + relevance scores for exploration and downstream use
  - does **not** create a physical table (compiled as a CTE into downstream models)

### Fact models
- **fct_ratings** (incremental)
  - appends only new rating rows based on latest `rating_timestamp`
  - avoids full rebuilds for large datasets
- **fct_genome_scores**
  - rounds relevance values (e.g., to 4 decimals)
  - filters invalid/zero relevance values (project choice)

### Mart models
- **mart_movie_releases**
  - joins curated facts/dims with seed reference data
  - creates a reporting-friendly dataset

---

## Materializations used
This project intentionally demonstrates multiple dbt materializations:
- **view** (default): lightweight, good for early modeling and iteration
- **table** (dims/facts): fast querying for BI use cases
- **incremental** (`fct_ratings`): efficient loads for growing event tables
- **ephemeral** (`dim_movies_with_tags`): avoids storing intermediate tables

You can see these configured in `dbt_project.yml`.

---

## Data quality: tests & validation
Testing is a core part of this repo. It includes:
- **not_null**: required keys and critical fields
- **unique**: primary keys in dims/seeds
- **relationships**: FK integrity between facts and dimensions
- **custom tests** (SQL in `tests/`): business-rule checks, e.g., relevance score constraints

Run tests with:
```bash
dbt test
```

---

## Snapshots (SCD Type 2 history)
Snapshots track changes to a mutable dataset over time, preserving history with `valid_from` / `valid_to` style columns.

This repo includes a snapshot:
- **snap_tags**: captures tag history over time based on `tag_timestamp` and a row-key

Run snapshots with:
```bash
dbt snapshot
```

---

## Seeds (reference data)
Seeds are small CSV files committed to the repo and loaded into the warehouse.
This project uses a seed to enrich reporting in the mart layer (release dates).

Load seeds with:
```bash
dbt seed
```

---

## How to run (end-to-end)
### 0) Preconditions
- You already loaded raw MovieLens tables into Snowflake (RAW schema)
- You have a working dbt profile locally (do **not** commit `profiles.yml`)

### 1) Install dependencies
```bash
dbt deps
```

### 2) Build models
```bash
dbt run
```

### 3) Run tests
```bash
dbt test
```

### 4) Run snapshots (optional)
```bash
dbt snapshot
```

### 5) Generate documentation and open lineage
```bash
dbt docs generate
dbt docs serve
```
Then open the local site (usually `http://localhost:8080`).

---

## How to review this repo quickly (for recruiters)
If you have 2 minutes:
1. Check `models/` folder: staging → dims → facts → marts
2. Open `models/schema.yml`: tests and documentation are defined there
3. Look at `models/fct/fct_ratings.sql`: incremental logic is implemented
4. Look at `snapshots/snap_tags.sql`: history tracking is implemented
5. Run `dbt docs` to see lineage and column-level docs

---

## Notes on security & best practices
- `profiles.yml` is intentionally **not included** (contains credentials)
- `target/`, `logs/`, `dbt_packages/`, and `venv/` should not be committed
- `.gitignore` helps prevent accidental uploads

---

## Potential improvements (future)
- Add CI (GitHub Actions) to run `dbt deps` + `dbt compile` on pull requests
- Publish dbt docs via GitHub Pages
- Add additional marts for KPI reporting (genre trends, user cohorts, etc.)
- Add exposures for BI dashboards and downstream consumers

---

## Author
**Kushal Jain**  
MS Information Systems (Business Analytics), FIU (Dec 2025)  
LinkedIn: https://www.linkedin.com/in/kushaljaink/
