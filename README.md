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

## Snowflake resources

### Databases & Schemas

Below is the canonical list of databases and schemas used in the platform. Adjust names if your org uses a different naming prefix.

* `RAW_DB`

  * `RAW_DB.public` — landing raw data (do not transform here)
  * `RAW_DB.metadata` — ingestion metadata and audit tables
* `BRONZE_DB`

  * `BRONZE_DB.events` — first-stage cleansed/normalized data
  * `BRONZE_DB.staging` — staging tables used by ETL jobs
* `SILVER_DB`

  * `SILVER_DB.transforms` — business logic transformations
  * `SILVER_DB.dimensions` — dimension tables
* `GOLD_DB`

  * `GOLD_DB.analytics` — curated tables for BI and ML
  * `GOLD_DB.aggr` — aggregated, materialized results

> Replace the `*_DB` placeholders with your environment prefix (e.g. `dev_raw`, `qa_bronze`, `prod_gold`) when provisioning per environment.

### Warehouses

Recommended warehouses for different workloads (example sizes are suggestions — tune to workload & cost):

* `WH_LOAD_SMALL` — size `X-SMALL`, auto-suspend `60s` — for small ingestion jobs
* `WH_LOAD_MEDIUM` — size `MEDIUM`, auto-suspend `120s`, multi-cluster auto-scale `OFF` — for regular batch loads
* `WH_TRANSFORM` — size `LARGE`, auto-suspend `300s`, auto-scale `ECONOMY` — for dbt/SQL transformations
* `WH_REPORTING` — size `X-LARGE`, auto-suspend `600s`, auto-scale `STANDARD` — for BI/concurrent queries

---

## Naming conventions

* Databases: `<env>_<tier>_db` (e.g. `prod_gold_db`)
* Schemas: short, descriptive (e.g. `staging`, `transforms`, `analytics`)
* Warehouses: `<env>_wh_<purpose>_<size>` (e.g. `dev_wh_transform_medium`)
* Roles: `role_<env>_<purpose>` (e.g. `role_prod_reader`)

Consistent naming makes automation and role policies simpler.

---

## Required roles & grants (high level)

* `ACCOUNTADMIN` (reserved — limited personnel)
* `SYSADMIN` — used for object creation and ownership
* `DATA_ENGINEER_ROLE` — owned by engineering team; needs create/usage on warehouses & databases
* `ETL_SERVICE_ROLE` — used by CICD/ETL tools; granted USAGE on DBs & OPERATE on specific warehouses
* `ANALYST_ROLE` — read-only on curated schemas/tables

Example minimal grants (conceptual):

```sql
GRANT USAGE ON DATABASE GOLD_DB TO ROLE ANALYST_ROLE;
GRANT SELECT ON ALL TABLES IN SCHEMA GOLD_DB.analytics TO ROLE ANALYST_ROLE;
GRANT USAGE ON WAREHOUSE WH_REPORTING TO ROLE ANALYST_ROLE;
```

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

## Environment / CI-CD notes

* Keep environment-specific names in your infra-as-code (Terraform/CloudFormation/Ansible) or in CI variables.
* CI/CD (e.g., GitHub Actions) should run provisioning under a service role with least privilege (ETL_SERVICE_ROLE).
* Store Snowflake credentials and key material in your secret manager (Vault/GCP Secret Manager/Secrets Manager).

Suggested GitHub Actions steps:

1. `validate` — lint SQL/dbt
2. `plan` — preview infra changes (if using Terraform)
3. `apply` — create DB/schema/warehouse using infra tool or run SQL as deployment user

---

## Best practices

* Use separate databases per environment (dev/qa/prod).
* Use auto-suspend to save costs.
* Monitor warehouse credits and set alerts (Aceldata / Snowflake Resource Monitors).
* Version-control DDL and keep migrations in the repo.
* Use role-based access control for service accounts.

---

## Troubleshooting & contacts

* If objects fail to create: check `SYSADMIN` ownership and your role privileges.
* For cost spikes: inspect `WAREHOUSE` usage history and queued queries.

**Owners/Contacts:**

* Data Engineering Team: `data-eng@example.com`
* Platform Owner: `platform@example.com`

---

If you want, I can also add environment-specific manifests (dev/prod SQL files), Terraform examples, or dbt `profiles.yml` snippets.
