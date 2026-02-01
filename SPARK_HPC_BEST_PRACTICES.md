# Spark on HPC: Best Practices for Resource Configuration

This guide explains how to properly configure Spark memory and cores when running on an HPC cluster like SDSC Expanse. Understanding these concepts helps you avoid common pitfalls and get the most out of your allocated resources.

---

## Key Takeaways

- **Driver memory should be small** (1-2GB)—it coordinates but doesn't process data
- **Maximize executor memory**—executors do the actual work
- **Executor instances** = total cores - 1 (reserve 1 core for the driver)
- **Executor memory** = (total memory - driver memory) / number of executors

---

## Understanding the Spark Architecture

### From Single-Machine to Distributed

You've seen how Python and Spark run on your laptop with limited resources. On an HPC cluster, Spark distributes work across multiple **executors**, each running on its own core with dedicated memory.

```
┌─────────────────────────────────────────────────────────┐
│                     Your Allocation                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐             │
│  │  Driver  │   │ Executor │   │ Executor │  ...        │
│  │  (1 core)│   │  (1 core)│   │  (1 core)│             │
│  │   1-2GB  │   │   18GB   │   │   18GB   │             │
│  └──────────┘   └──────────┘   └──────────┘             │
│       ↑              ↑              ↑                   │
│       └──────────────┴──────────────┘                   │
│              Spark Cluster Manager                      │
└─────────────────────────────────────────────────────────┘
```

**Key components:**
- **Driver**: Coordinates the job, builds execution plans, schedules tasks
- **Executors**: Perform the actual data processing in parallel

---

## Why Keep Driver Memory Small?

### What the Driver Actually Does

The driver's responsibilities require minimal memory:
- **Builds the DAG** (execution plan)—just metadata, not data
- **Schedules tasks**—tracking task status is lightweight
- **Receives small results**—aggregates like `count()`, `sum()`, `mean()`

### What the Driver Does NOT Do

The driver doesn't process your data:
- It doesn't hold partitions of your DataFrame
- It doesn't execute transformations like `map()`, `filter()`, `join()`
- It doesn't store intermediate shuffle data

**Bottom line:** 1-2GB is sufficient for most workloads. Giving the driver more memory wastes resources that could go to executors.

---

## The Formula

```
Driver memory = 1-2GB (fixed, small)
Executor instances = Total Cores - 1
Executor memory = (Total Memory - Driver Memory) / Executor Instances
```

---

## Configuration Examples

### Example 1: 8 Cores with 16GB RAM

**Calculation:**
- Driver: **2GB** (fixed)
- Remaining: 16GB - 2GB = **14GB**
- Executors: 8 - 1 = **7 instances**
- Per executor: 14GB / 7 = **2GB each**

```python
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "2g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

**When to use:** Development, testing, small datasets (< 10GB)

### Example 2: 16 Cores with 32GB RAM

**Calculation:**
- Driver: **2GB** (fixed)
- Remaining: 32GB - 2GB = **30GB**
- Executors: 16 - 1 = **15 instances**
- Per executor: 30GB / 15 = **2GB each**

```python
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "2g") \
    .config("spark.executor.instances", 15) \
    .getOrCreate()
```

**When to use:** Medium datasets (10-50GB), more parallelism needed

### Example 3: 8 Cores with 128GB RAM (High Memory)

**Calculation:**
- Driver: **2GB** (fixed)
- Remaining: 128GB - 2GB = **126GB**
- Executors: 8 - 1 = **7 instances**
- Per executor: 126GB / 7 = **18GB each**

```python
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "18g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

**When to use:** Large datasets (50GB+), memory-intensive joins/aggregations

---

## Quick Reference Table

| Cores | Total Memory | Driver | Executors | Executor Memory |
|-------|--------------|--------|-----------|-----------------|
| 4 | 8GB | 2GB | 3 | 2GB |
| 8 | 16GB | 2GB | 7 | 2GB |
| 8 | 32GB | 2GB | 7 | 4GB |
| 8 | 64GB | 2GB | 7 | 8GB |
| 8 | 128GB | 2GB | 7 | 18GB |
| 16 | 32GB | 2GB | 15 | 2GB |
| 16 | 64GB | 2GB | 15 | 4GB |
| 16 | 128GB | 2GB | 15 | 8GB |
| 32 | 128GB | 2GB | 31 | 4GB |
| 64 | 250GB | 2GB | 63 | 4GB |
| 128 | 250GB | 2GB | 127 | 2GB |

---

## Driver Memory: When to Increase Beyond 2GB

### Rare Cases Requiring More Driver Memory

Only increase driver memory when you're explicitly bringing large data to the driver:

```python
# These operations bring data TO the driver - may need more driver memory
df.collect()           # Entire DataFrame to driver - AVOID on large data!
df.toPandas()          # Converts to Pandas on driver - AVOID on large data!
df.take(1000000)       # Large take operations
spark.sparkContext.broadcast(large_dict)  # Broadcasting large lookup tables

# These do NOT require extra driver memory
df.write.parquet()     # Data stays distributed
df.count()             # Only a single number returned
df.show(20)            # Only 20 rows returned
df.describe().show()   # Only summary statistics returned
```

**Rule of thumb:** If you're using `collect()` or `toPandas()` on large results, you're probably doing something wrong. Write to Parquet instead.

---

## Trade-offs: Low vs High Driver Memory

Understanding the trade-offs helps you make informed decisions when exceptions arise.

| Aspect | Low Driver Memory (1-2GB) | High Driver Memory (4GB+) |
|--------|---------------------------|---------------------------|
| **Executor memory** | Maximized—more for data processing | Reduced—less for actual work |
| **Large collects** | Will fail with OOM errors | Can handle larger results |
| **Broadcast variables** | Limited size broadcasts | Larger lookup tables possible |
| **Resource efficiency** | Optimal for most workloads | Wasteful if driver doesn't need it |
| **Complex DAGs** | May struggle with 1000+ stages | Better for very complex plans |
| **Best for** | Production ETL, batch processing | Interactive analysis, ML pipelines |

### Advantages of Low Driver Memory ✓

1. **Maximizes processing power**—executors get more memory for actual data work
2. **Encourages best practices**—forces you to avoid anti-patterns like large `collect()`
3. **Efficient resource usage**—no wasted memory sitting idle on the driver
4. **Scales better**—works consistently regardless of cluster size

### Disadvantages of Low Driver Memory ✗

1. **Less flexibility**—can't easily collect results for quick inspection
2. **Broadcast limitations**—can't broadcast very large lookup tables
3. **Complex job failures**—extremely complex DAGs may fail to compile

---

## ⚠️ CAVEATS: When to Deviate from Best Practices

The following scenarios may require larger driver memory. **These are exceptions, not the norm.** If you find yourself needing these frequently, consider whether your approach could be improved.

### Caveat 1: Interactive Analysis in Jupyter Notebooks

**Scenario:** You're exploring data interactively and need to bring results to Pandas for visualization.

```python
# Interactive exploration pattern - may need 4-8GB driver
sample_df = df.sample(0.01).toPandas()  # 1% sample to Pandas
plt.scatter(sample_df['x'], sample_df['y'])
```

**Why this deviates:** Interactive work often requires collecting results for plotting or inspection. Sampling mitigates this, but you may still need more driver memory.

**Recommendation:** Use 4GB driver for interactive work. Still prioritize executor memory.

```python
# Interactive configuration (8 cores, 128GB)
spark = SparkSession.builder \
    .config("spark.driver.memory", "4g") \
    .config("spark.executor.memory", "17g") \  # (128-4)/7 ≈ 17GB
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

### Caveat 2: Broadcasting Large Lookup Tables

**Scenario:** You need to join a large DataFrame with a smaller lookup table (e.g., 1-2GB of reference data).

```python
# Broadcasting a large lookup table
large_lookup = spark.read.parquet("reference_data.parquet")  # ~2GB
broadcast_lookup = spark.sparkContext.broadcast(
    large_lookup.collect()  # This goes to driver first!
)
```

**Why this deviates:** The driver must hold the entire broadcast variable in memory before distributing it to executors.

**Recommendation:** If your broadcast variable is > 1GB, increase driver memory accordingly. Better alternative: use a broadcast join instead of manual broadcast.

```python
# Better: Let Spark handle the broadcast join
from pyspark.sql.functions import broadcast
result = large_df.join(broadcast(small_lookup_df), "key")
```

### Caveat 3: Machine Learning Model Collection

**Scenario:** Training ML models that need to be collected to the driver for saving or further processing.

```python
# Collecting trained models
from pyspark.ml.classification import RandomForestClassifier
model = rf.fit(training_data)
# Model parameters are collected to driver
model.save("path/to/model")
```

**Why this deviates:** Some ML operations aggregate model parameters on the driver. Large ensemble models (many trees, deep networks) may require more driver memory.

**Recommendation:** For ML workloads, consider 4GB driver memory as a starting point.

### Caveat 4: Very Complex Execution Plans

**Scenario:** Jobs with thousands of stages, deeply nested transformations, or complex SQL queries.

```python
# Complex query with many joins and subqueries
result = (df1
    .join(df2, "key1")
    .join(df3, "key2")
    .join(df4, "key3")
    # ... many more joins
    .groupBy("category")
    .agg(complex_aggregations)
    .filter(complex_conditions))
```

**Why this deviates:** The driver compiles the execution plan (DAG). Extremely complex plans with 1000+ stages may require more memory for plan optimization.

**Recommendation:** This is rare. If you hit this, simplify your query first. Only increase driver memory if simplification isn't possible.

### Caveat 5: Reading Many Small Files

**Scenario:** Reading thousands of small files (common with streaming data or log files).

```python
# Reading 10,000+ small files
df = spark.read.parquet("logs/year=2024/month=*/day=*/hour=*/*.parquet")
```

**Why this deviates:** The driver tracks metadata for each file/partition. With tens of thousands of files, this metadata can consume significant memory.

**Recommendation:** Compact small files into larger ones before processing. If not possible, increase driver memory to 4GB.

---

## Decision Matrix: Choosing Driver Memory

Use this matrix to determine when to deviate from the 2GB default:

| Your Workload | Driver Memory | Justification |
|---------------|---------------|---------------|
| Batch ETL (read → transform → write) | **2GB** | Standard best practice |
| Interactive Jupyter analysis | **4GB** | Frequent small collects for visualization |
| ML training pipelines | **4GB** | Model parameter aggregation |
| Jobs with broadcast joins > 1GB | **4-8GB** | Broadcast variable storage |
| Extremely complex queries (1000+ stages) | **4GB** | DAG compilation overhead |
| Processing 10,000+ small files | **4GB** | File metadata tracking |

**Default to 2GB** unless you have a specific reason from the table above.

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Giving Driver Too Much Memory

```python
# BAD: Driver gets 16GB when it only needs 2GB
spark = SparkSession.builder \
    .config("spark.driver.memory", "16g") \
    .config("spark.executor.memory", "16g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

**Problem:** With 8 cores and 128GB, this allocates 16GB to the driver that could go to executors. You're wasting 14GB.

**Fix:** Keep driver at 1-2GB, give the rest to executors:

```python
# GOOD: Driver gets 2GB, executors get the rest
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "18g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

### Mistake 2: Forgetting to Reserve a Core for Driver

```python
# BAD: 8 executors on 8 cores leaves no room for driver
spark = SparkSession.builder \
    .config("spark.executor.instances", 8) \  # Should be 7!
    .getOrCreate()
```

**Problem:** Resource contention causes slowdowns or failures.

**Fix:** Always use `total_cores - 1` for executor instances.

### Mistake 3: Mismatched Jupyter Request and Spark Config

If you request 8 cores and 16GB in Jupyter but configure Spark for 16GB per executor:

```python
# BAD: Requested 16GB total, but configuring 16GB per executor
spark = SparkSession.builder \
    .config("spark.executor.memory", "16g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

**Problem:** Spark will fail to allocate executors—you're asking for 112GB when you only have 16GB.

**Fix:** Your Spark config must fit within your Jupyter allocation.

### Mistake 4: Too Many Small Executors

```python
# SUBOPTIMAL: 128 executors with 2GB each
spark = SparkSession.builder \
    .config("spark.executor.memory", "2g") \
    .config("spark.executor.instances", 127) \
    .getOrCreate()
```

**Problem:** Overhead per executor adds up. JVM and Spark internals consume ~300MB-1GB per executor.

**Better:** Fewer executors with more memory when doing memory-intensive work.

---

## Choosing Your Configuration

### Start Small, Scale Up

1. **Development**: Start with 4 cores, 8GB
2. **Testing**: Move to 8 cores, 32GB
3. **Production**: Scale to 16+ cores, 64GB+ based on needs

### Decision Tree

```
Is your data > 100GB?
├── Yes → Use 64+ cores, 128GB+ RAM
└── No
    ├── Are you doing many joins/aggregations?
    │   ├── Yes → Prioritize memory (8 cores, 128GB)
    │   └── No → Balance (16 cores, 64GB)
    └── Is it exploratory/development?
        └── Yes → Start small (8 cores, 16GB)
```

---

## Memory Overhead Consideration

Spark reserves additional memory beyond what you configure for internal operations. The default overhead is:

```
spark.executor.memoryOverhead = max(384MB, 0.10 × executor.memory)
```

For a 16GB executor:
- Overhead: 1.6GB
- Available for processing: ~14.4GB

**Practical impact:** If you're hitting memory limits, you may need slightly more memory than your data size suggests.

---

## Summary

### Key Concepts

1. **Driver memory should be small** (1-2GB)—it coordinates but doesn't process data
2. **Maximize executor memory**—give executors the remaining memory after the driver
3. **Reserve 1 core** for the driver; remaining cores run executors
4. **Start small** and scale up based on actual needs
5. **Match your Spark config** to your Jupyter/SLURM allocation

### Configuration Template

```python
# Template: Adjust TOTAL_MEMORY and TOTAL_CORES to match your allocation
TOTAL_MEMORY = 128  # GB - from your Jupyter request
TOTAL_CORES = 8     # from your Jupyter request
DRIVER_MEMORY = 2   # GB - fixed, small (driver doesn't process data)

num_executors = TOTAL_CORES - 1
executor_memory = (TOTAL_MEMORY - DRIVER_MEMORY) // num_executors

spark = SparkSession.builder \
    .config("spark.driver.memory", f"{DRIVER_MEMORY}g") \
    .config("spark.executor.memory", f"{executor_memory}g") \
    .config("spark.executor.instances", num_executors) \
    .getOrCreate()
```

---

## Understanding Shuffle and Communication Costs

### What is a Shuffle?

A **shuffle** occurs when Spark redistributes data across executors. Shuffles are the most expensive operations in Spark because they involve:

1. **Disk I/O**: Writing intermediate data to local disk
2. **Network I/O**: Transferring data between executors
3. **Serialization**: Converting objects to bytes and back

### Operations That Cause Shuffles

| Operation | Shuffles? | Alternative |
|-----------|-----------|-------------|
| `groupBy()` | Yes | Use `reduceByKey()` when possible |
| `join()` on large tables | Yes | Use `broadcast()` for small tables |
| `repartition(n)` | Yes | Use `coalesce(n)` to reduce partitions |
| `distinct()` | Yes | Use `dropDuplicates(subset)` on key columns |
| `orderBy()` | Yes | Avoid unless needed for output |

### Minimizing Shuffle Costs

**1. Use Broadcast Joins for Small Tables**

```python
from pyspark.sql.functions import broadcast

# BAD: Both tables shuffle
result = large_df.join(small_df, "key")

# GOOD: Small table broadcast to all executors (no shuffle)
result = large_df.join(broadcast(small_df), "key")
```

**2. Filter Early**

```python
# BAD: Shuffle all data, then filter
result = df.groupBy("category").count().filter(col("count") > 100)

# GOOD: Filter before shuffle (if possible)
result = df.filter(col("value").isNotNull()).groupBy("category").count()
```

**3. Use Coalesce Instead of Repartition**

```python
# BAD: Full shuffle to reduce partitions
df.repartition(10).write.parquet("output")

# GOOD: No shuffle, just combine partitions locally
df.coalesce(10).write.parquet("output")
```

### Identifying Shuffles in Spark UI

In the Spark UI:
- Each **stage boundary** represents a shuffle
- Check **Shuffle Read/Write** in the Stages tab
- Look for **Spill (Memory/Disk)** indicating memory pressure

**Red Flags in Spark UI:**
- Shuffle Spill > 0: Consider more memory per executor
- One task much slower than others: Data skew
- Shuffle Read >> Shuffle Write: Data explosion during shuffle

### For Your Project

In your README.md, document:
1. Number of shuffles in your pipeline
2. Shuffle data volume (from Spark UI)
3. Any optimizations you made to reduce shuffles

See the full [Communication Costs Guide](../Class16/07_communication_costs.md) for detailed examples.

---

## Measuring Speedup

Understanding how well your code parallelizes is essential. Measure speedup to validate your distributed implementation.

### Speedup Formula

$$\text{Speedup} = \frac{T_1}{T_n}$$

Where $T_1$ is time with 1 executor and $T_n$ is time with n executors.

### Efficiency

$$\text{Efficiency} = \frac{\text{Speedup}}{n}$$

| Efficiency | Interpretation |
|------------|----------------|
| > 80% | Excellent parallelization |
| 50-80% | Good, some overhead |
| < 50% | Significant bottlenecks |

### How to Measure

```python
import time

# Baseline: 1 executor
start = time.time()
result = df.groupBy("key").count().collect()
T_1 = time.time() - start

# Parallel: n executors
start = time.time()
result = df.groupBy("key").count().collect()
T_n = time.time() - start

speedup = T_1 / T_n
efficiency = speedup / n_executors
print(f"Speedup: {speedup:.2f}x, Efficiency: {efficiency:.1%}")
```

### For Your Project (Milestone 3)

Include a speedup analysis table:

| Executors | Time (sec) | Speedup | Efficiency |
|-----------|------------|---------|------------|
| 1         | X          | 1.00x   | 100%       |
| 7         | Y          | X/Y     | (X/Y)/7    |

See the full [Speedup Measurement Guide](../Class16/06_speedup_measurement.md) for detailed instructions.

---

## Further Reading

- [Spark Configuration Guide](https://spark.apache.org/docs/latest/configuration.html)
- [Tuning Spark](https://spark.apache.org/docs/latest/tuning.html)
- [SDSC Expanse User Guide](https://www.sdsc.edu/support/user_guides/expanse.html)
- [Speedup Measurement Guide](../Class16/06_speedup_measurement.md)
- [Communication Costs Guide](../Class16/07_communication_costs.md)
- [Spark UI Debugging Lab](../Class16/08_spark_ui_debugging.md)
- [Spark vs Ray Comparison](../Class16/09_framework_comparison.md)

---

*This guide is part of DSC 232R: Big Data Analysis Using Spark at UCSD.*
