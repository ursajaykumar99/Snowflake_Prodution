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

* EDW_PRODUCTION_DB

 * EDW_PROD_LANDING_SC

 * EDW_PROD_BRONZE_SC

 * EDW_PROD_SILVER_SC

 * EDW_PROD_GOLD_SC

Development

* EDW_DEVELOPMENT_DB

 * EDW_DEV_LANDING_SC

 * EDW_DEV_BRONZE_SC

 * EDW_DEV_SILVER_SC

 * EDW_DEV_GOLD_SC

Quality Analysis (QA)

* EDW_QUALITY_ANALYSIS_DB

 * EDW_QA_LANDING_SC

 * EDW_QA_BRONZE_SC

 * EDW_QA_SILVER_SC

 * EDW_QA_GOLD_SC

## Warehouses

Recommended warehouses for different workloads (example sizes are suggestions — tune to workload & cost):

* EDW_PROD_SMALL_WH — size X-SMALL, auto-suspend 60s — for small ingestion jobs

* EDW_PROD_MEDIUM_WH — size MEDIUM, auto-suspend 120s, multi-cluster auto-scale OFF — for regular batch loads

* EDW_PROD_LARGE_WH— size LARGE, auto-suspend 300s, auto-scale ECONOMY — for dbt/SQL transformations

* EDW_PROD_XLARGE_WH — size X-LARGE, auto-suspend 600s, auto-scale STANDARD — for BI/concurrent queries

---

## Naming conventions

* Databases: `EDW_<USAGE>_DB` (e.g. `EDW_PRODUCTION_DB`)
* Schemas: `EDW_<USAGE>_<LAYER>_SC` (e.g. `EDW_PROD_LANDING_SC`)
* Warehouses: `EDW_<USAGE>_<SIZE>_WH` (e.g. `EDW_PROD_MEDIUM_WH`)
* Roles: `role_<env>_<purpose>` (e.g. `role_prod_reader`)

Consistent naming makes automation and role policies simpler.

---

## Quick setup — SQL snippets

Create a database, schema and a warehouse (example):

```sql
CREATE DATABASE IF NOT EXISTS EDW_PRODUCTION_DB;
CREATE SCHEMA IF NOT EXISTS EDW_PRODUCTION_DB.EDW_PROD_LANDING_SC;

CREATE WAREHOUSE IF NOT EXISTS EDW_PROD_LARGE_WH
  WITH WAREHOUSE_SIZE = 'LARGE'
  WAREHOUSE_TYPE = 'STANDARD'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE;

```
