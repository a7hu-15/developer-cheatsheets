# ⚡ Apache Spark & Data Engineering Cheatsheet

A practical reference guide for PySpark, DataFrame transformations, shuffle performance optimization, memory management, and Delta Lake ACID transactions.

---

## 🏎️ Core Architecture & Execution Model

```
[ Driver Process ]
   ├── SparkContext / SparkSession
   ├── DAGScheduler (Splits job into Stages across shuffle boundaries)
   └── TaskScheduler (Schedules tasks onto Executor slots)
            │
            ▼
[ Worker Nodes / Executors ]
   ├── Task Slot 1 (Processes 1 RDD/DataFrame Partition)
   ├── Task Slot 2
   └── Storage Memory / Execution Memory Pool
```

- **Narrow Transformations**: No data shuffle across network (e.g. `map()`, `filter()`, `withColumn()`, `select()`). Executed pipelined within the same stage.
- **Wide Transformations**: Triggers network shuffle and stage boundary (e.g. `groupBy()`, `join()`, `reduceByKey()`, `distinct()`).

---

## 🐍 PySpark DataFrame Essentials

### Initialization & Reading Data
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.window import Window

spark = SparkSession.builder \
    .appName("DataEngineeringPipeline") \
    .config("spark.sql.shuffle.partitions", "200") \
    .config("spark.driver.memory", "4g") \
    .getOrCreate()

# Read Parquet / CSV / JSON
df = spark.read.parquet("s3a://data-lake/raw/events/*/")
```

### Common Transformations
```python
# Filtering, column creation, conditional logic
df_cleaned = df \
    .filter(F.col("status") == "COMPLETED") \
    .withColumn("amount_usd", F.col("amount_cents") / 100) \
    .withColumn(
        "customer_tier",
        F.when(F.col("total_spend") > 1000, "PLATINUM")
         .when(F.col("total_spend") > 500, "GOLD")
         .otherwise("SILVER")
    )

# Aggregation
df_agg = df_cleaned.groupBy("customer_tier").agg(
    F.count("order_id").alias("total_orders"),
    F.avg("amount_usd").alias("avg_order_value"),
    F.max("created_at").alias("latest_order_time")
)
```

### Window Functions
```python
# Calculate running total and dense rank per customer
window_spec = Window.partitionBy("customer_id").orderBy(F.col("order_timestamp").asc())

df_windowed = df.withColumn(
    "running_total", F.sum("amount_usd").over(window_spec)
).withColumn(
    "order_rank", F.dense_rank().over(window_spec)
)
```

---

## 🚀 Performance Optimization & Tuning

### 1. Broadcast Joins (Avoid Shuffling Large + Small Datasets)
When joining a large table with a small dimension table (< 10MB default), broadcast the small table to eliminate network shuffle.

```python
# Explicit broadcast hint
df_joined = df_fact.join(F.broadcast(df_dim_customer), "customer_id", "left")
```

### 2. Adaptive Query Execution (AQE) - Spark 3.x+
Enables runtime query re-optimization:
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true") # Merges small shuffle partitions
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")           # Handles skewed join keys
```

### 3. Repartitioning vs Coalescing
- `df.repartition(n)`: Wide transformation. Performs full network shuffle to increase or decrease partition count evenly.
- `df.coalesce(n)`: Narrow transformation. Combines existing adjacent partitions without a full shuffle (use only when reducing partitions).

---

## 🛢️ Delta Lake ACID Transactions

```python
# Write DataFrame as Delta Lake table with schema enforcement
df.write.format("delta") \
    .mode("overwrite") \
    .partitionBy("event_date") \
    .save("s3a://data-lake/delta/events_table")

# Time Travel Query
df_v1 = spark.read.format("delta").option("versionAsOf", 1).load("s3a://data-lake/delta/events_table")
```

---

## 📋 Best Practices Checklist
1. **Filter Early**: Apply `.filter()` as close to the data source read as possible to leverage predicate pushdown.
2. **Avoid UDFs**: Native PySpark SQL functions (`F.col`, `F.when`) run in optimized C++/JVM bytecode via Catalyst optimizer; Python UDFs serialize data back and forth to Python runtime.
3. **Persist Wisely**: Use `df.persist(StorageLevel.MEMORY_AND_DISK)` only when a DataFrame is referenced multiple times across branching actions. Always call `df.unpersist()` afterwards.
