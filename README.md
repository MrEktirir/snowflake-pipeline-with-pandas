# Data Engineering Pipelines with pandas on Snowflake

![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Engineering-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)
![Python](https://img.shields.io/badge/Python-Snowpark-3776AB?style=for-the-badge&logo=python&logoColor=white)

## Overview

This repository contains my implementation of the Snowflake Quickstart **Data Engineering Pipelines with pandas on Snowflake**.

The project demonstrates how to build a data engineering pipeline using **pandas on Snowflake (Snowpark pandas)** with the Snowflake Sample TPC-H dataset. The pipeline transforms transactional order data into customer-level features, persists the resulting customer profile in Snowflake, and operationalizes the workflow using a Python Stored Procedure and a scheduled Snowflake Task.

The repository is based on the official Snowflake Quickstart and has been used as a hands-on implementation to explore Snowpark pandas, feature engineering, pipeline orchestration, and task execution in Snowflake.

## Architecture

```text
SNOWFLAKE_SAMPLE_DATA.TPCH_SF1
        │
        ├── LINEITEM
        │     ├── Column selection
        │     ├── Filtering
        │     ├── Feature engineering
        │     └── Aggregation
        │
        └── ORDERS
              │
              ▼
        LEFT JOIN
              │
              ▼
     Customer Aggregation
              │
              ▼
       Customer Profile
              │
              ├── Snowflake Table
              │
              └── Python Stored Procedure
                         │
                         ▼
                  Serverless Task
                         │
                         ▼
              Scheduled Pipeline Runs
```

## Pipeline Workflow

The implementation follows the pipeline below:

1. Read `LINEITEM` and `ORDERS` from the Snowflake Sample TPC-H dataset using Snowpark pandas.
2. Select relevant columns and filter line items based on `L_RETURNFLAG`.
3. Create a derived `DISCOUNT_AMOUNT` feature.
4. Aggregate line-item data by order and return flag.
5. Join the aggregated line-item data with order information.
6. Aggregate order information by customer to generate customer-level features:
   - Number of orders
   - Total order amount
   - Average order amount
   - Median order amount
   - Total discount amount
7. Persist the resulting customer profile as a Snowflake table.
8. Package the transformation logic as a permanent Python Stored Procedure.
9. Create and schedule a Snowflake Task to execute the pipeline automatically.
10. Validate scheduled executions using Snowflake Task History and verify generated output tables.

## Customer Profile

The final customer-level dataset contains the following features:

| Feature | Description |
|---|---|
| `O_CUSTKEY` | Customer identifier |
| `NUMBER_OF_ORDERS` | Number of aggregated order records |
| `TOT_ORDER_AMOUNT` | Total order amount |
| `AVG_ORDER_AMOUNT` | Average order amount |
| `MEDIAN_ORDER_AMOUNT` | Median order amount |
| `TOT_DISCOUNT_AMOUNT` | Total calculated discount amount |

During the implementation, the resulting customer profile contained **99,996 customer records**.

## Orchestration

The transformation logic is registered as a permanent Snowflake Python Stored Procedure:

```text
CREATE_CUSTOMER_PROFILE_SP
```

A Snowflake Task executes the Stored Procedure on a schedule:

```text
CREATE_CUSTOMER_PROFILE_TASK
        │
        ▼
CREATE_CUSTOMER_PROFILE_SP
        │
        ▼
Snowpark pandas transformations
        │
        ▼
CUSTOMER_PROFILE_<timestamp>
```

For the Quickstart implementation, the task was scheduled at one-minute intervals to validate repeated pipeline execution.

Task execution was verified through `INFORMATION_SCHEMA.TASK_HISTORY`, and successful runs generated timestamped customer profile tables.

After validation, the task was suspended and the generated test tables were cleaned up.

## Key Engineering Concepts

This project provided hands-on experience with:

- Snowpark pandas for in-Snowflake dataframe transformations
- Filtering, feature engineering, joins, and aggregations
- Data grain awareness during aggregation
- Materializing Snowpark pandas results into Snowflake tables
- Moving from Snowpark pandas to native pandas with `to_pandas()`
- Python Stored Procedures in Snowflake
- Snowflake Serverless Tasks
- Scheduled data pipeline execution
- Task lifecycle management
- Pipeline observability using Task History
- Programmatic Snowflake object management with the Snowflake Python API

## Snowpark pandas vs Native pandas

Most transformations are executed using Snowpark pandas so that processing remains within Snowflake.

Native pandas is only used after the dataset has been reduced to the customer-profile level:

```text
Large TPC-H tables
        │
        ▼
Snowpark pandas
        │
        ├── Filter
        ├── Join
        ├── Feature Engineering
        └── Aggregate
        │
        ▼
Reduced Customer Profile
        │
        ▼
to_pandas()
        │
        ▼
Native pandas
```

This demonstrates an important data engineering pattern: perform large-scale transformations close to the data and materialize data locally only after reducing the dataset to an appropriate size.

## Implementation Notes

The Quickstart was implemented using the current Snowflake Notebook environment rather than reproducing older package versions exactly.

During the implementation:

- Snowpark pandas transformations and table persistence executed successfully.
- The customer profile was successfully materialized to native pandas.
- Matplotlib/Seaborn inline visualization did not render in the current Snowflake Notebook environment, although the underlying pandas dataset was successfully materialized and validated.
- The scheduled pipeline was successfully deployed, executed, monitored, suspended, and cleaned up.

## Technologies

- Snowflake
- Snowpark Python
- Snowpark pandas / Modin
- Python
- Snowflake Python API
- Python Stored Procedures
- Snowflake Tasks
- TPC-H Sample Data
- pandas
- Matplotlib
- Seaborn

## Source

This repository is a fork and hands-on implementation of the official Snowflake Quickstart:

**Data Engineering Pipelines with pandas on Snowflake**

The original step-by-step tutorial, prerequisites, and setup instructions are available in the [Snowflake Quickstart Guide](https://quickstarts.snowflake.com/guide/data_engineering_pipelines_with_snowpark_pandas/index.html).

Additional documentation:

- [pandas on Snowflake](https://docs.snowflake.com/developer-guide/snowpark/python/snowpark-pandas)
- [Snowflake Sample TPC-H Data](https://docs.snowflake.com/en/user-guide/sample-data-tpch)