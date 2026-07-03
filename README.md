# Enterprise DWH pipeline on Microsoft Fabric — architecture write-up

This documents the design of a data warehouse pipeline I built on Microsoft Fabric. The production pipeline definitions and notebooks belong to the company I built it for, so this repo is the architecture write-up rather than runnable code.

## The flow

Source systems (SQL Server, PostgreSQL, Oracle, REST APIs) feed a four-stage pipeline orchestrated with Fabric Data Factory:

1. **Source → Landing** — raw data copied into a Lakehouse landing zone with no transformation. Tables land prefixed by source system so you always know where a record came from.
2. **Landing → Dimensions** — PySpark notebooks apply business rules and load the dimension tables (customers, products, dates, locations), with surrogate keys and SCD handling.
3. **Landing → Facts** — transactional data joined against the freshly loaded dimension keys, measures calculated, fact tables loaded.
4. **Audit** — a `LAST_REFRESH` table gets a row per run (pipeline name, timestamp with UTC+3 offset, status, record counts), which feeds the monitoring dashboard and answers "how fresh is this data?" without asking the data team.

## Dependency logic

- Stage 1 → 2 runs only on successful ingestion
- Stage 2 → 3 runs on completion (not just success) so fact loading is always attempted
- Stage 3 → 4 runs only on success, so the audit timestamp never lies about a half-loaded warehouse

## Load strategy

First run is a full historical load of every source table. After that, incremental refresh pulls only new and changed records directly from the source systems on a schedule. Getting the incremental watermarks right per source (timestamp columns vs. IDs, different databases behaving differently) was most of the real work.

## Stack

Microsoft Fabric (Data Factory, Lakehouse, Warehouse, PySpark notebooks), SQL, star schema / Kimball-style dimensional modeling.
