# 05 - Performance and Best Practices

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Performance factors affecting COPY INTO
- Small File Problem
- File Size Recommendations
- Cluster Sizing
- Storage Optimization
- COPY INTO Scheduling
- Cost Optimization
- Production Best Practices
- Common Mistakes
- Associate Exam Tips
- Professional Interview Questions

---

# Introduction

Although COPY INTO is a simple SQL command, its performance depends on several design decisions.

Poor pipeline design can lead to:

- Long execution times
- Higher cloud costs
- Slow batch completion
- Underutilized compute
- Excessive file operations

A well-designed COPY INTO pipeline minimizes cost while maximizing throughput.

---

# Performance Factors

COPY INTO performance depends on:

```
Performance

│

├── Number of Files

├── File Size

├── Cluster Configuration

├── Storage Throughput

├── Parallel Processing

├── File Format

└── Network Latency
```

Improving these factors can significantly reduce ingestion time.

---

# Small File Problem

One of the biggest performance bottlenecks in distributed systems is processing a very large number of small files.

Example:

```
100 GB

↓

100 Files

≈ Fast
```

vs

```
100 GB

↓

1,000,000 Files

≈ Slow
```

Although both datasets contain the same amount of data, the second requires much more overhead.

---

# Why Small Files Are Slow

For every file, Spark performs:

```
List File

↓

Open File

↓

Read Metadata

↓

Read Data

↓

Close File
```

When millions of files exist, most execution time is spent managing files rather than reading data.

---

# Recommended File Sizes

Although requirements vary, a common recommendation is:

```
100 MB

↓

1 GB
```

Benefits:

- Fewer file operations
- Better parallelism
- Lower metadata overhead
- Faster ingestion

Avoid thousands of files smaller than a few megabytes whenever possible.

---

# Parallel Processing

COPY INTO leverages Apache Spark to process multiple files concurrently.

Example

```
sales/

001.csv

002.csv

003.csv

004.csv
```

Execution

```
Worker 1 → 001.csv

Worker 2 → 002.csv

Worker 3 → 003.csv

Worker 4 → 004.csv
```

Instead of processing files sequentially, Spark distributes them across worker nodes.

---

# Cluster Sizing

Increasing cluster size is **not always** the solution.

Example

```
2 Million Files

↓

5 KB Each
```

Even with a large cluster, performance may remain poor because the workload is dominated by file management overhead.

Better solutions include:

- Combine small files upstream.
- Deliver fewer, larger files.
- Optimize storage layout.

---

# File Format Performance

Different file formats have different performance characteristics.

| File Format | Performance | Reason |
|--------------|------------|--------|
| Parquet | ⭐⭐⭐⭐⭐ | Columnar, compressed |
| ORC | ⭐⭐⭐⭐⭐ | Columnar, compressed |
| Avro | ⭐⭐⭐⭐ | Binary format |
| JSON | ⭐⭐⭐ | Text parsing required |
| CSV | ⭐⭐ | Text parsing, no schema |

Whenever possible, prefer **Parquet** for production pipelines.

---

# Storage Optimization

Best practices:

- Keep compute and storage in the same cloud region.
- Organize files using logical directory structures.
- Archive processed files.
- Avoid unnecessary cross-region reads.

Example

```
Bronze/

2026/

08/

01/

sales.csv
```

A clear directory structure simplifies maintenance and improves operational efficiency.

---

# Scheduling COPY INTO

COPY INTO is designed for **batch processing**.

Typical schedules:

```
Every Hour

Every Day

Every Week

Every Month
```

Scheduling tools:

- Databricks Jobs
- Apache Airflow
- Azure Data Factory
- AWS Step Functions

Choose the schedule based on business requirements.

---

# COPY INTO vs Auto Loader Performance

| COPY INTO | Auto Loader |
|------------|-------------|
| Batch | Continuous |
| Runs only when scheduled | Continuously monitors storage |
| Lower compute cost for infrequent loads | Better for frequent arrivals |
| Simple architecture | Streaming architecture |

### Example

Daily finance reports:

```
Every Midnight

↓

COPY INTO

↓

Cluster Stops
```

IoT sensors sending data every minute:

```
Every Minute

↓

Auto Loader

↓

Streaming
```

---

# Cost Optimization

Recommendations:

✅ Use Job Clusters instead of All-Purpose Clusters.

✅ Stop compute after ingestion.

✅ Deliver larger files.

✅ Avoid unnecessary FORCE reloads.

✅ Schedule ingestion according to SLAs.

---

# Production Architecture

```
ERP

↓

Cloud Storage

↓

Databricks Job

↓

COPY INTO

↓

Bronze

↓

Silver

↓

Gold

↓

Dashboards
```

This architecture is common for nightly ETL pipelines.

---

# Monitoring

Monitor:

- Job duration
- Files processed
- Failed loads
- Cluster utilization
- Storage latency
- Data volume trends

Monitoring helps identify bottlenecks before they affect downstream systems.

---

# Best Practices

✅ Keep Bronze append-only.

✅ Use immutable source files.

✅ Generate unique filenames.

✅ Deliver reasonably large files.

✅ Validate files before loading.

✅ Archive processed files.

✅ Keep business logic out of COPY INTO.

---

# Common Mistakes

❌ Running COPY INTO every minute for streaming workloads.

❌ Uploading millions of tiny files.

❌ Oversizing clusters to solve small-file problems.

❌ Mixing ingestion with transformations.

❌ Reusing filenames for corrected data.

---

# Associate Exam Notes

Remember:

```
COPY INTO

↓

Batch

----------------

Auto Loader

↓

Continuous

----------------

Small Files

↓

Slow

----------------

Large Files

↓

Fast

----------------

Job Clusters

↓

Lower Cost
```

---

# Professional Interview Questions

## Question 1

A COPY INTO job is slow because it processes 3 million CSV files.

Would increasing the cluster size solve the problem?

### Expected Answer

Not necessarily.

The primary bottleneck is file management overhead (listing, opening, and closing files). The better solution is to reduce the number of files by combining them into larger files before ingestion.

---

## Question 2

Why are Parquet files generally faster than CSV files?

### Expected Answer

Parquet is a columnar storage format that supports compression and efficient column pruning. CSV is a text format that requires parsing every row and column.

---

## Question 3

When would you choose COPY INTO over Auto Loader?

### Expected Answer

When data arrives at predictable intervals (hourly, daily, weekly) and batch processing is sufficient. COPY INTO avoids the cost of a continuously running streaming job.

---

## Question 4

How can you reduce the cost of COPY INTO pipelines?

### Expected Answer

- Use Job Clusters.
- Stop clusters after execution.
- Optimize file sizes.
- Schedule jobs only when required.
- Avoid unnecessary reloads using `FORCE = TRUE`.

---

# Chapter Summary

```
Cloud Storage

↓

COPY INTO

↓

Bronze

↓

Silver

↓

Gold
```

Performance depends on:

- File size
- Number of files
- Cluster sizing
- Storage layout
- Scheduling strategy

---

# Key Takeaways

- COPY INTO performance is heavily affected by the number of files.
- Small files create significant metadata and file management overhead.
- Prefer files between **100 MB and 1 GB** for most workloads.
- Use Job Clusters for scheduled ingestion to reduce costs.
- Keep COPY INTO focused on ingestion; perform transformations in downstream layers.
- Monitor pipeline performance and optimize storage layout for production systems.