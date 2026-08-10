# 02 - COPY INTO Architecture and Internals

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Internal Architecture of COPY INTO
- Execution Flow
- Metadata Tracking
- Idempotency
- Exactly-Once File Loading
- FORCE = TRUE
- Failure Scenarios
- Corrected File Handling
- Production Architecture
- Best Practices

---

# Introduction

At first glance, COPY INTO appears to be a simple SQL command.

```sql
COPY INTO bronze.orders
FROM 's3://company-data/orders/'
FILEFORMAT = CSV;
```

However, internally Databricks performs several operations before loading data.

Unlike a normal INSERT statement, COPY INTO maintains metadata about processed files to ensure that the same file is not loaded multiple times.

---

# High-Level Architecture

```
Cloud Storage

↓

COPY INTO

↓

List Files

↓

Check Load Metadata

↓

Identify New Files

↓

Read Files

↓

Write Delta Table

↓

Commit

↓

Update Metadata
```

The most important step is **checking the load metadata** before reading files.

---

# Internal Components

```
COPY INTO

│

├── File Discovery

├── Metadata Manager

├── File Reader

├── Delta Writer

└── Commit Manager
```

Each component performs a specific task.

---

# Step 1 – File Discovery

COPY INTO begins by scanning the configured source path.

Example:

```
orders/

001.csv

002.csv

003.csv
```

It creates a list of available files.

---

# Step 2 – Metadata Lookup

Next, COPY INTO checks its internal load metadata.

Example:

```
Already Loaded

001.csv

002.csv
```

Incoming files:

```
001.csv

002.csv

003.csv
```

Comparison:

| File | Metadata Exists? | Action |
|------|------------------|--------|
| 001.csv | Yes | Skip |
| 002.csv | Yes | Skip |
| 003.csv | No | Load |

Only **003.csv** will be processed.

---

# Step 3 – Read New Files

COPY INTO reads only files that are not present in its metadata.

```
New File

↓

Read File

↓

Parse Data
```

Supported formats include:

- CSV
- JSON
- Parquet
- ORC
- Avro
- Text

---

# Step 4 – Write to Delta

After reading the file, COPY INTO writes the data into the target Delta table.

```
Read File

↓

Delta Writer

↓

Bronze Table
```

At this point, the data has not yet been marked as successfully loaded.

---

# Step 5 – Commit Transaction

Once the write completes successfully, Delta Lake commits the transaction.

```
Write Complete

↓

Commit Transaction
```

Only after a successful commit does COPY INTO record the file as processed.

---

# Step 6 – Update Metadata

```
Commit Successful

↓

Update Load Metadata
```

This ordering is critical.

Metadata is **never updated before a successful commit**.

---

# Why Update Metadata After Commit?

Imagine the opposite sequence.

```
Read File

↓

Update Metadata

↓

Cluster Crash

↓

Write Never Completed
```

Next execution:

```
Metadata Exists

↓

Skip File
```

Result:

```
Data Loss
```

The file is skipped even though it was never written.

To avoid this, COPY INTO follows this sequence:

```
Read File

↓

Write Data

↓

Commit

↓

Update Metadata
```

If a failure occurs before the commit, the metadata is not updated, allowing the file to be safely retried.

---

# Idempotency

COPY INTO is **idempotent at the file level**.

Running the same command multiple times produces the same result because previously processed files are skipped.

Example:

```
Run 1

↓

001.csv

002.csv

Loaded
```

Run again:

```
001.csv

↓

Already Loaded

↓

Skip

002.csv

↓

Already Loaded

↓

Skip
```

No duplicate records are inserted.

---

# Failure Scenario

Suppose the following happens.

```
Read 004.csv

↓

Write Starts

↓

Cluster Failure
```

Since the transaction was never committed, metadata is not updated.

On the next execution:

```
004.csv

↓

Metadata Missing

↓

Load Again
```

The file is processed successfully.

This behavior prevents data loss.

---

# FORCE = TRUE

By default:

```sql
FORCE = FALSE
```

COPY INTO skips files that already exist in its metadata.

Example:

```
001.csv

↓

Metadata Exists

↓

Skip
```

---

When:

```sql
FORCE = TRUE
```

COPY INTO ignores the metadata.

```
001.csv

↓

Load Again
```

This forces the file to be processed again.

---

# Why Use FORCE = TRUE?

Typical scenarios:

- Historical backfill
- Intentional data reload
- Testing
- Recovery after manual table cleanup

---

# Risks of FORCE = TRUE

Suppose:

```
001.csv

↓

Already Loaded
```

Running

```sql
FORCE = TRUE
```

results in:

```
001.csv

↓

Load Again

↓

Duplicate Records
```

Unless downstream deduplication exists, duplicate rows will be inserted.

---

# Corrected File Scenario

Suppose

```
orders.csv
```

contains incorrect data.

The upstream team fixes the file but keeps the same filename.

```
orders.csv

↓

Corrected Data
```

Running COPY INTO normally:

```
Metadata Exists

↓

Skip
```

The corrected file is **not** loaded.

Possible solutions:

1. Rename the corrected file (recommended).
2. Use `FORCE = TRUE` and deduplicate downstream.
3. Remove or rebuild the affected data if appropriate.

---

# Enterprise Architecture

```
ERP System

↓

Amazon S3

↓

COPY INTO

↓

Bronze Delta Table

↓

Silver

↓

Gold
```

COPY INTO is typically scheduled using Databricks Jobs, Apache Airflow, Azure Data Factory, or similar orchestration tools.

---

# Best Practices

✅ Use immutable source files.

✅ Keep filenames unique.

✅ Use `FORCE = TRUE` only when necessary.

✅ Load into the Bronze layer.

✅ Perform deduplication in the Silver layer if reprocessing is required.

---

# Common Mistakes

❌ Assuming COPY INTO overwrites existing data.

❌ Deleting source files immediately after loading without retention policies.

❌ Using `FORCE = TRUE` without understanding duplicate risks.

❌ Reusing filenames for corrected data.

---

# Associate Exam Notes

Remember:

```
List Files

↓

Check Metadata

↓

Read New Files

↓

Write Delta

↓

Commit

↓

Update Metadata
```

Key points:

- Metadata is checked before reading.
- Metadata is updated after a successful commit.
- COPY INTO is idempotent by default.
- `FORCE = TRUE` bypasses metadata checks.

---

# Professional Interview Questions

## Question 1

Why is COPY INTO considered idempotent?

**Expected Answer**

Because it tracks previously loaded files using internal metadata. Files that have already been successfully ingested are skipped during subsequent executions, preventing duplicate file ingestion.

---

## Question 2

Why is metadata updated only after a successful commit?

**Expected Answer**

Updating metadata before the commit could cause data loss if the write fails. By updating metadata only after the transaction commits successfully, COPY INTO guarantees reliable retries without skipping unwritten files.

---

## Question 3

When would you use `FORCE = TRUE`?

**Expected Answer**

For intentional file reprocessing, historical backfills, or testing. It should be used carefully because it bypasses metadata checks and may introduce duplicate records.

---

## Question 4

A source system fixes a file but keeps the same filename.

What happens?

**Expected Answer**

COPY INTO skips the file because its filename already exists in the load metadata. The recommended approach is to upload the corrected data with a new filename or intentionally reload using `FORCE = TRUE` while handling duplicates downstream.

---

# Chapter Summary

```
Cloud Storage

↓

List Files

↓

Check Metadata

↓

Read New Files

↓

Write Delta Table

↓

Commit

↓

Update Metadata
```

COPY INTO provides reliable, idempotent file ingestion by maintaining load metadata and updating it only after successful commits.

---

# Key Takeaways

- COPY INTO tracks previously loaded files using internal metadata.
- Metadata is checked before reading files.
- Metadata is updated only after a successful commit.
- COPY INTO is idempotent by default.
- `FORCE = TRUE` bypasses metadata checks and reloads files.
- Renaming corrected files is generally safer than relying on `FORCE = TRUE`.