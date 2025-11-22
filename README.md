# Snowflake Data Engineering Project

> **Purpose:** Central repository for Snowflake objects (Databases, Schemas, Warehouses) and setup instructions used by the data engineering pipelines.

---

## Table of contents

* [Overview](#overview)
* [Snowflake resources](#snowflake-resources)

  * [Databases & Schemas](#databases--schemas)
  * [Warehouses](#warehouses)
* [Naming conventions](#naming-conventions)
* [Required roles & grants (high level)](#required-roles--grants-high-level)
* [Quick setup — SQL snippets](#quick-setup--sql-snippets)
* [Environment / CI-CD notes](#environment--ci-cd-notes)
* [Best practices](#best-practices)
* [Troubleshooting & contacts](#troubleshooting--contacts)

---

## Overview

This project documents the Snowflake footprint required for the data engineering pipelines powering the Bronze → Silver → Gold architecture. It contains the definitions and example SQL for creating the required **Databases**, **Schemas**, and **Warehouses** used by ingestion, transformation, and analytics.

Use this README as the single source of truth for on-boarding, provisioning, and code-first infra changes.

---

Snowflake resources
Databases & Schemas

Below are the databases and schemas currently provisioned for the EDW environments:

Production

EDW_PRODUCTION_DB

EDW_PROD_LANDING_SC

EDW_PROD_BRONZE_SC

EDW_PROD_SILVER_SC

EDW_PROD_GOLD_SC

Development

EDW_DEVELOPMENT_DB

EDW_DEV_LANDING_SC

EDW_DEV_BRONZE_SC

EDW_DEV_SILVER_SC

EDW_DEV_GOLD_SC

Quality Analysis (QA)

EDW_QUALITY_ANALYSIS_DB

EDW_QA_LANDING_SC

EDW_QA_BRONZE_SC

EDW_QA_SILVER_SC

EDW_QA_GOLD_SC

Warehouses

Recommended warehouses for different workloads (example sizes are suggestions — tune to workload & cost):

WH_LOAD_SMALL — size X-SMALL, auto-suspend 60s — for small ingestion jobs

WH_LOAD_MEDIUM — size MEDIUM, auto-suspend 120s, multi-cluster auto-scale OFF — for regular batch loads

WH_TRANSFORM — size LARGE, auto-suspend 300s, auto-scale ECONOMY — for dbt/SQL transformations

WH_REPORTING — size X-LARGE, auto-suspend 600s, auto-scale STANDARD — for BI/concurrent queries

---

## Naming conventions

* Databases: `<env>_<tier>_db` (e.g. `prod_gold_db`)
* Schemas: short, descriptive (e.g. `staging`, `transforms`, `analytics`)
* Warehouses: `<env>_wh_<purpose>_<size>` (e.g. `dev_wh_transform_medium`)
* Roles: `role_<env>_<purpose>` (e.g. `role_prod_reader`)

Consistent naming makes automation and role policies simpler.

---

## Quick setup — SQL snippets

Create a database, schema and a warehouse (example):

```sql
CREATE DATABASE IF NOT EXISTS BRONZE_DB;
CREATE SCHEMA IF NOT EXISTS BRONZE_DB.staging;

CREATE WAREHOUSE IF NOT EXISTS WH_TRANSFORM
  WITH WAREHOUSE_SIZE = 'LARGE'
  WAREHOUSE_TYPE = 'STANDARD'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 1;

-- Grant usage to roles
GRANT USAGE ON DATABASE BRONZE_DB TO ROLE DATA_ENGINEER_ROLE;
GRANT USAGE ON WAREHOUSE WH_TRANSFORM TO ROLE DATA_ENGINEER_ROLE;
```

Add schema/table ownership and secure views according to your governance policy.

---

If you want, I can also add environment-specific manifests (dev/prod SQL files), Terraform examples, or dbt `profiles.yml` snippets.
