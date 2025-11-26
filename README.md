# Snowflake Data Engineering Project

> **Purpose:** Central repository for Snowflake objects (Databases, Schemas, Warehouses) and setup instructions used by the data engineering pipelines.

---

## Table of contents

* [Overview](#overview)
* [Snowflake resources](#snowflake-resources)

  * [Databases & Schemas](#databases--schemas)
  * [Warehouses](#warehouses)
* [Naming conventions](#naming-conventions)
* [Quick setup — SQL snippets](#quick-setup--sql-snippets)

---

## Overview

This project documents the Snowflake footprint required for the data engineering pipelines powering the Bronze → Silver → Gold architecture. It contains the definitions and example SQL for creating the required **Databases**, **Schemas**, and **Warehouses** used by ingestion, transformation, and analytics.

---

## Snowflake resources

## Databases & Schemas

Below are the databases and schemas currently provisioned for the EDW environments:

Production

* `EDW_PRODUCTION_DB`

 * `EDW_PROD_LANDING_SC`
 * `EDW_PROD_BRONZE_SC`
 * `EDW_PROD_SILVER_SC`
 * `EDW_PROD_GOLD_SC`
 * `EDW_PROD_PROCESS_SC`

Development

* `EDW_DEVELOPMENT_DB`

 * `EDW_DEV_LANDING_SC`
 * `EDW_DEV_BRONZE_SC`
 * `EDW_DEV_SILVER_SC`
 * `EDW_DEV_GOLD_SC`
 * `EDW_DEV_PROCESS_SC`

Quality Analysis (QA)

* `EDW_QUALITY_ANALYSIS_DB`

 * `EDW_QA_LANDING_SC`
 * `EDW_QA_BRONZE_SC`
 * `EDW_QA_SILVER_SC`
 * `EDW_QA_GOLD_SC`
 * `EDW_QA_PROCESS_SC`

## Warehouses

Recommended warehouses for different workloads (example sizes are suggestions — tune to workload & cost):

* `EDW_PROD_SMALL_WH` — size SMALL, auto-suspend 300s — for small ingestion jobs
* `EDW_PROD_MEDIUM_WH` — size MEDIUM, auto-suspend 300s  — for regular batch loads
* `EDW_PROD_HEAVYLOAD_WH`— size MEDIUM, auto-suspend 300s, multi-cluster auto-scale STANDARD — for dbt/SQL transformations
* `EDW_DEV_SMALL_WH` — size SMALL, auto-suspend 300s — for DEVELOPMENT
* `EDW_QA_SMALL_WH` — size SMALL, auto-suspend 300s — for QA

---

## Naming conventions

* Databases: `EDW_<USAGE>_DB` (e.g. `EDW_PRODUCTION_DB`)
* Schemas: `EDW_<USAGE>_<LAYER>_SC` (e.g. `EDW_PROD_LANDING_SC`)
* Warehouses: `EDW_<USAGE>_<SIZE>_WH` (e.g. `EDW_PROD_MEDIUM_WH`)
* Roles: `ETL_<purpose>_RL` (e.g. `ETL_DEVELOPER_RL`)

Consistent naming makes automation and role policies simpler.

---

## Quick setup — SQL snippets

Create a database, schema, warehouse, users and roles (example).

```sql
CREATE DATABASE EDW_PRODUCTION_DB;
CREATE SCHEMA EDW_PRODUCTION_DB.EDW_PROD_LANDING_SC;

CREATE OR REPLACE WAREHOUSE EDW_PROD_HEAVYLOAD_WH
  WAREHOUSE_SIZE = 'MEDIUM'
  WAREHOUSE_TYPE = 'STANDARD'
  MIN_CLUSTER_COUNT = 2
  MAX_CLUSTER_COUNT = 5
  SCALING_POLICY = 'STANDARD'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;

CREATE ROLE ETL_DEVELOPER_RL;
CREATE ROLE ETL_TESTER_RL;

GRANT USAGE ON DATABASE EDW_PRODUCTION_DB TO ROLE ETL_DEVELOPER_RL;
GRANT USAGE ON DATABASE EDW_DEVELOPMENT_DB TO ROLE ETL_DEVELOPER_RL;
GRANT USAGE ON ALL SCHEMAS IN DATABASE EDW_PRODUCTION_DB TO ROLE ETL_DEVELOPER_RL;
GRANT USAGE ON ALL SCHEMAS IN DATABASE EDW_DEVELOPMENT_DB TO ROLE ETL_DEVELOPER_RL;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE EDW_PRODUCTION_DB TO ROLE ETL_DEVELOPER_RL;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE EDW_DEVELOPMENT_DB TO ROLE ETL_DEVELOPER_RL;

CREATE USER AJAYKUMARKARRI
  PASSWORD = 'Temp_Pass#123'
  DEFAULT_ROLE = ETL_DEVELOPER_RL
  MUST_CHANGE_PASSWORD = TRUE;

CREATE USER RENUSRIBOTTA
  PASSWORD = 'Temp_Pass#123'
  DEFAULT_ROLE = ETL_DEVELOPER_RL
  MUST_CHANGE_PASSWORD = TRUE;

GRANT ROLE ETL_DEVELOPER_RL TO USER AJAYKUMARKARRI;
GRANT ROLE ETL_DEVELOPER_RL TO USER RENUSRIBOTTA;

```
