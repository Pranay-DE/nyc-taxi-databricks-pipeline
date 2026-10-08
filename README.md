# NYC Taxi Databricks Pipeline

End-to-end **PySpark + Delta Lake** pipeline on NYC TLC trip data, built entirely inside Databricks.

## Overview

- **Ingestion:** Raw TLC parquet files uploaded to a Databricks Volume, loaded as Bronze Delta tables
- **Transformation:** PySpark DataFrames shaping Bronze → Silver → Gold
- **Storage:** Delta Lake (native to Databricks)
- **Orchestration:** Databricks Workflows (native scheduler)
- **Governance:** Unity Catalog

## Architecture

```mermaid
flowchart LR
    A[TLC Parquet Files] --> B[Bronze Delta Tables]
    B --> C[Silver Cleaned Tables]
    C --> D[Gold Business Tables]
    D --> E[Dashboards / BI]
```

## Tech Stack

- **Databricks** — Lakehouse platform
- **PySpark** — distributed data processing
- **Delta Lake** — ACID storage layer
- **Databricks Workflows** — orchestration
- **Unity Catalog** — governance and lineage

## Dataset

NYC TLC Trip Record Data — [source](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

## Status

🚧 Project in progress — Gold layer complete

### Completed
- ✅ Databricks catalog, schemas (`bronze`/`silver`/`gold`), and Volume created
- ✅ 3 months of NYC TLC Yellow Taxi data (April–June 2025) uploaded to Volume
- ✅ Bronze Delta table `nyc_taxi.bronze.raw_trips` created (12,885,358 rows, 19 columns)
- ✅ Data quality audit — see [`docs/DATA_QUALITY_AUDIT.md`](./docs/DATA_QUALITY_AUDIT.md)
- ✅ Audit notebook: `notebooks/02_silver_audit.ipynb`
- ✅ Silver Delta table `nyc_taxi.silver.trips_clean` created (11,634,221 rows after cleaning and deduplication)
- ✅ Silver validation — all impossible-record checks pass (0 invalid rows)
- ✅ Derived columns added: `trip_duration_minutes`, `tip_percentage`, `pickup_hour`, `pickup_day_of_week`, `has_valid_passenger_count`
- ✅ Gold layer — 4 tables:
  - `gold.dim_date` — Kimball date dimension (91 rows)
  - `gold.daily_summary` — daily trips + revenue aggregates (91 rows)
  - `gold.hourly_demand` — demand by day-of-week × hour (168 rows)
  - `gold.zone_performance` — trips + revenue by pickup zone, joined with zone lookup (261 rows)

### Next Steps
- 🔜 Databricks Workflows — schedule the pipeline to run daily
- 🔜 Dashboard / BI — visualize the Gold tables

## 📈 Results

**Bronze ingestion — 12.88M rows loaded into a Delta table:**

![Bronze ingestion](docs/databricks_bronze_ingestion.png)

**Data quality audit — anomalies found:**

![Audit — dates](docs/silver_audit_dates.png)
![Audit — invalid data](docs/silver_audit_invalid_data.png)

**Silver validation — impossible records removed (all checks pass):**

![Silver validation](docs/silver_validation.png)

**Silver validation — duplicate check returns 0:**

![Silver validation duplicates](docs/silver_validation_duplicates.png)

> 📓 Full validation notebook: [`notebooks/04_silver_validation.ipynb`](./notebooks/04_silver_validation.ipynb)

**Gold layer — all 4 tables built and verified:**
![Gold verification](docs/gold_verification.png)