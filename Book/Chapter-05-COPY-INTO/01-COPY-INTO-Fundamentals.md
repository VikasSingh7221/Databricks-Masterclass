# 01 - COPY INTO Fundamentals

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- What is COPY INTO?
- Why COPY INTO was introduced
- Problems with traditional file ingestion
- COPY INTO Architecture
- COPY INTO Workflow
- Supported File Formats
- Supported Cloud Storage
- COPY INTO vs INSERT
- COPY INTO vs Auto Loader
- Enterprise Use Cases
- Best Practices

---

# Introduction

In every data engineering project, the first step is to ingest data from an external source into the Lakehouse.

Typical sources include:

- CSV files
- JSON files
- Parquet files
- Avro files
- ORC files
- Application logs
- ERP exports
- CRM systems

These files are commonly stored in cloud object storage such as Amazon S3, Azure Data Lake Storage (ADLS), or Google Cloud Storage (GCS).

Before any transformations can occur, the data must be loaded into a Delta table.

Databricks provides **COPY INTO** as a simple, reliable, and idempotent SQL command for batch file ingestion.

---

# What is COPY INTO?

COPY INTO is a SQL command that loads data from cloud object storage into an existing Delta table.

Unlike a simple INSERT statement, COPY INTO keeps track of previously loaded files and avoids loading the same file again.

This makes COPY INTO **idempotent** and safe to execute multiple times.

---

# Definition

> **COPY INTO is a SQL-based batch ingestion command that loads files from cloud object storage into Delta tables while automatically tracking previously loaded files to prevent duplicate ingestion.**

---

# Why Was COPY INTO Introduced?

Before COPY INTO, developers often built custom ingestion scripts.

Typical workflow:

```
List Files

↓

Identify New Files

↓

Read Files

↓

Insert into Table

↓

Maintain Metadata Manually
```

Problems:

- Custom code required
- Manual metadata management
- Duplicate file handling
- Complex retry logic
- Higher maintenance effort

COPY INTO automates these tasks.

---

# The Problem with Traditional Batch Loading

Imagine a folder containing:

```
sales/

001.csv

002.csv

003.csv
```

A batch job loads these files today.

Tomorrow, a new file arrives.

```
sales/

001.csv

002.csv

003.csv

004.csv
```

Without tracking processed files, the batch job may reload all four files, creating duplicate records.

Developers had to write custom logic to prevent this.

COPY INTO solves this automatically.

---

# COPY INTO Architecture

```
Cloud Storage

↓

COPY INTO

↓

Check File Metadata

↓

Read New Files

↓

Write Delta Table

↓

Update Load Metadata
```

Notice the important step:

**COPY INTO checks previously loaded file metadata before reading files.**

---

# COPY INTO Workflow

Suppose your storage contains:

```
001.csv

002.csv

003.csv
```

### First Execution

```
COPY INTO

↓

Read 001

↓

Read 002

↓

Read 003

↓

Write Table

↓

Store Metadata
```

The Delta table now contains data from all three files.

---

### Second Execution

Storage contains:

```
001.csv

002.csv

003.csv

004.csv
```

COPY INTO checks its metadata.

```
001

Already Loaded

↓

Skip

----------------

002

Already Loaded

↓

Skip

----------------

003

Already Loaded

↓

Skip

----------------

004

New File

↓

Load
```

Only **004.csv** is processed.

---

# Supported Cloud Storage

COPY INTO supports:

- Amazon S3
- Azure Data Lake Storage (ADLS)
- Google Cloud Storage (GCS)

Example:

```
s3://company-data/orders/

abfss://sales@storage.dfs.core.windows.net/orders/

gs://company-data/orders/
```

---

# Supported File Formats

COPY INTO supports several file formats.

| Format | Supported |
|----------|-----------|
| CSV | ✅ |
| JSON | ✅ |
| Parquet | ✅ |
| Avro | ✅ |
| ORC | ✅ |
| Text | ✅ |

---

# Basic Syntax

```sql
COPY INTO bronze.orders
FROM 's3://company-data/orders/'
FILEFORMAT = CSV;
```

This command:

1. Reads files from the specified path.
2. Loads new files into the Delta table.
3. Records metadata for processed files.

---

# COPY INTO vs INSERT

Many beginners confuse these commands.

### INSERT

```
INSERT INTO table

VALUES(...)
```

Characteristics:

- Inserts rows directly.
- Does not read files.
- Does not track loaded files.

---

### COPY INTO

```
Cloud Storage

↓

Read Files

↓

Load Table
```

Characteristics:

- Reads external files.
- Tracks loaded files.
- Prevents duplicate file ingestion.

---

# COPY INTO vs Auto Loader

This is a common certification question.

## COPY INTO

Designed for:

- Batch ingestion
- Historical data loading
- Scheduled jobs

Example:

```
Every Night

↓

COPY INTO

↓

Job Ends
```

---

## Auto Loader

Designed for:

- Continuous ingestion
- Near real-time file arrival
- Streaming pipelines

Example:

```
New File Arrives

↓

Automatically Detected

↓

Loaded
```

---

# Comparison

| Feature | COPY INTO | Auto Loader |
|----------|-----------|-------------|
| Processing | Batch | Continuous |
| Streaming | No | Yes |
| Monitoring | Manual/Scheduled | Automatic |
| Best Use Case | Historical or scheduled loads | Continuously arriving files |
| Metadata Tracking | Yes | Yes |

---

# Enterprise Example

Suppose a retail company receives a daily sales report.

```
Every Night

↓

sales_2026_08_01.csv

↓

COPY INTO

↓

Bronze Table
```

The next day:

```
sales_2026_08_02.csv

↓

COPY INTO

↓

Only New File Loaded
```

This pattern is ideal for daily batch processing.

---

# Advantages of COPY INTO

✅ Simple SQL syntax

✅ Automatic metadata tracking

✅ Prevents duplicate file ingestion

✅ Supports multiple file formats

✅ Easy to schedule

✅ Ideal for historical and batch loads

---

# Limitations

COPY INTO is **not** designed for:

- Continuous streaming ingestion
- Event-driven processing
- Millisecond latency

For these scenarios, Auto Loader is the recommended solution.

---

# Best Practices

✅ Use COPY INTO for batch pipelines.

✅ Load raw data into the Bronze layer.

✅ Schedule COPY INTO using Databricks Jobs or orchestration tools.

✅ Keep source files immutable after loading.

---

# Common Mistakes

❌ Using COPY INTO for continuously arriving files.

❌ Assuming COPY INTO overwrites existing data.

❌ Deleting and recreating source files with the same name without understanding metadata behavior.

❌ Performing heavy business transformations during ingestion.

---

# Associate Exam Notes

Remember:

```
COPY INTO

↓

Batch Ingestion

--------------------

Auto Loader

↓

Continuous Ingestion

--------------------

INSERT

↓

Insert Rows

Not Files
```

---

# Professional Interview Questions

## Question 1

Why would you choose COPY INTO over Auto Loader?

**Expected Answer**

COPY INTO is ideal for one-time or scheduled batch ingestion where files arrive at known intervals. It is simple to configure, idempotent, and does not require a continuously running streaming job.

---

## Question 2

How does COPY INTO prevent duplicate file ingestion?

**Expected Answer**

COPY INTO maintains metadata about previously loaded files. During subsequent executions, it checks this metadata and skips files that have already been successfully ingested.

---

## Question 3

Should COPY INTO be used for continuously arriving files every few seconds?

**Expected Answer**

No. Auto Loader is a better choice because it is designed for continuous, event-driven file ingestion using Structured Streaming.

---

# Chapter Summary

```
Cloud Storage

↓

COPY INTO

↓

Check Metadata

↓

Read New Files

↓

Write Delta Table

↓

Update Metadata
```

COPY INTO provides reliable, idempotent batch ingestion into Delta Lake.

---

# Key Takeaways

- COPY INTO is a SQL command for loading files into Delta tables.
- It automatically tracks previously loaded files.
- It supports multiple cloud storage providers and file formats.
- It is best suited for batch ingestion and historical data loading.
- Auto Loader should be used instead for continuous file ingestion.