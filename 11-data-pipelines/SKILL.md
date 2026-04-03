---
name: data-pipelines
description: Auto-activating data pipeline skill — Airflow DAGs, dbt models, Spark jobs, Kafka streams, Flink, Beam, data quality checks, CDC, and pipeline monitoring
origin: jeremylongshore/claude-code-plugins-plus-skills/11-data-pipelines
tools: Read, Write, Edit, Bash, Grep
---

# Data Pipelines Skill

Automated assistance for data engineering: orchestration (Airflow/Prefect/Dagster), transformation (dbt/Spark), streaming (Kafka/Flink), and data quality.

## When to Activate

Activates automatically when you mention: Airflow, DAG, dbt, Spark, Kafka streams, Flink, Beam, Prefect, Luigi, Dagster, data pipeline, ETL, ELT, CDC, data quality, schema validation, data lineage, data catalog, pipeline monitoring.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `airflow-dag-generator` | Airflow DAG with operators, dependencies, retries |
| `airflow-operator-creator` | Custom operators and sensors |
| `dagster-pipeline-creator` | Dagster assets, jobs, schedules |
| `prefect-flow-builder` | Prefect flows with tasks and deployments |
| `dbt-model-generator` | dbt models, refs, sources, incremental |
| `dbt-test-creator` | dbt tests (schema, custom, great expectations) |
| `spark-job-creator` | PySpark transformations with partitioning |
| `spark-sql-optimizer` | Catalyst optimizer hints, broadcast joins |
| `pyspark-transformer` | DataFrame transformations, UDFs |
| `kafka-stream-processor` | Kafka Streams / Faust consumer patterns |
| `flink-job-creator` | Flink DataStream API patterns |
| `beam-pipeline-builder` | Apache Beam Python pipeline |
| `sql-transform-helper` | SQL transformation patterns (CTEs, window functions) |
| `data-quality-checker` | Great Expectations / Soda Core checks |
| `schema-validator` | Avro/JSON Schema/Protobuf validation |
| `cdc-pipeline-creator` | Debezium CDC pipeline setup |
| `data-lineage-tracker` | OpenLineage integration |
| `incremental-load-setup` | Watermark-based incremental loading |
| `pipeline-monitoring-setup` | Alerting on SLA, data freshness, row counts |

## Airflow DAG Template

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.utils.task_group import TaskGroup

default_args = {
    "owner": "data-team",
    "depends_on_past": False,
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "email_on_failure": True,
}

with DAG(
    dag_id="etl_pipeline",
    default_args=default_args,
    description="Daily ETL pipeline",
    schedule="0 6 * * *",
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=["etl", "daily"],
    max_active_runs=1,
) as dag:

    def extract(**context):
        hook = PostgresHook(postgres_conn_id="source_db")
        df = hook.get_pandas_df("SELECT * FROM events WHERE date = '{{ ds }}'")
        context["ti"].xcom_push(key="row_count", value=len(df))
        df.to_parquet(f"/tmp/events_{{ ds }}.parquet")

    def transform(**context):
        import pandas as pd
        df = pd.read_parquet(f"/tmp/events_{{ ds }}.parquet")
        df = df.dropna(subset=["user_id"]).assign(
            processed_at=datetime.utcnow()
        )
        df.to_parquet(f"/tmp/events_transformed_{{ ds }}.parquet")

    with TaskGroup("extract_load") as tg:
        extract_task = PythonOperator(task_id="extract", python_callable=extract)
        transform_task = PythonOperator(task_id="transform", python_callable=transform)
        extract_task >> transform_task
```

## dbt Model (Incremental)

```sql
-- models/marts/fact_orders.sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge',
    on_schema_change='sync_all_columns'
) }}

WITH source AS (
    SELECT * FROM {{ ref('stg_orders') }}
    {% if is_incremental() %}
    WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
    {% endif %}
),
final AS (
    SELECT
        order_id,
        customer_id,
        SUM(amount) AS total_amount,
        COUNT(*) AS item_count,
        MIN(created_at) AS order_created_at,
        MAX(updated_at) AS updated_at
    FROM source
    GROUP BY 1, 2
)
SELECT * FROM final
```

## PySpark Job

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, DoubleType

spark = SparkSession.builder \
    .appName("EventAggregation") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .getOrCreate()

df = spark.read.parquet("s3://bucket/events/date={{ ds }}/")

result = df \
    .filter(F.col("event_type") == "purchase") \
    .groupBy("user_id", F.date_trunc("day", "timestamp").alias("date")) \
    .agg(
        F.sum("amount").alias("total_spend"),
        F.count("*").alias("purchase_count"),
        F.countDistinct("product_id").alias("unique_products")
    ) \
    .repartition(100)

result.write.mode("overwrite").partitionBy("date").parquet("s3://bucket/aggregated/")
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `11-data-pipelines` (26 skills)
