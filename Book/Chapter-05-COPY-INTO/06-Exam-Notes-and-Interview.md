# 06 - Exam Notes and Interview Guide

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# COPY INTO Cheat Sheet

## Definition

COPY INTO is a SQL command used to load data from cloud object storage into Delta tables while automatically tracking previously loaded files to prevent duplicate ingestion.

---

# COPY INTO Lifecycle

```
Cloud Storage

↓

List Files

↓

Check Load Metadata

↓

Read New Files

↓

Write Delta Table

↓

Commit Transaction

↓

Update Metadata
```

Golden Rule

> **Metadata is updated only after a successful commit.**

---

# Internal Workflow

```
Cloud Storage

↓

COPY INTO

↓

File Discovery

↓

Metadata Lookup

↓

Read Files

↓

Write Delta

↓

Commit

↓

Update Metadata
```

---

# COPY INTO Components

```
COPY INTO

│

├── File Discovery

├── Metadata Manager

├── File Reader

├── Delta Writer

└── Commit Manager
```

---

# Supported File Formats

| Format | Supported |
|---------|-----------|
| CSV | ✅ |
| JSON | ✅ |
| PARQUET | ✅ |
| AVRO | ✅ |
| ORC | ✅ |
| TEXT | ✅ |

---

# Supported Cloud Storage

- Amazon S3
- Azure Data Lake Storage (ADLS)
- Google Cloud Storage (GCS)

---

# COPY INTO vs INSERT

| INSERT | COPY INTO |
|----------|-----------|
| Inserts rows | Loads files |
| No metadata | Metadata tracking |
| No duplicate protection | Idempotent |
| Table data only | Cloud storage files |

---

# COPY INTO vs Auto Loader

| COPY INTO | Auto Loader |
|------------|-------------|
| Batch | Continuous |
| SQL Command | Structured Streaming |
| Scheduled | Long Running |
| Historical Loads | Continuous Ingestion |
| No Streaming | Streaming |

---

# Idempotency

Running COPY INTO repeatedly produces the same result.

Example

```
Run 1

↓

001.csv

↓

Loaded

----------------

Run 2

↓

001.csv

↓

Skipped
```

---

# Metadata Tracking

Metadata stores information about files that have already been successfully loaded.

Example

```
001.csv

002.csv

003.csv
```

Next execution

```
001.csv

↓

Skip

002.csv

↓

Skip

004.csv

↓

Load
```

---

# Exactly-Once Loading

Correct Order

```
Read File

↓

Write Delta

↓

Commit

↓

Update Metadata
```

Wrong Order

```
Read File

↓

Update Metadata

↓

Write Delta
```

This could lead to permanent data loss.

---

# FORCE = TRUE

Default

```sql
FORCE = FALSE
```

Behavior

```
Metadata Exists

↓

Skip
```

With

```sql
FORCE = TRUE
```

```
Ignore Metadata

↓

Reload File
```

---

# FILES vs PATTERN

## FILES

```
jan.csv

feb.csv
```

Known files.

---

## PATTERN

```
sales_2026.*

↓

Regex
```

Dynamic file selection.

---

# FORMAT_OPTIONS

Controls how files are read.

Examples

```
header

delimiter

quote

escape

nullValue

dateFormat

timestampFormat

multiLine
```

---

# COPY_OPTIONS

Controls loading behavior.

Examples

```
FORCE

mergeSchema
```

---

# mergeSchema

Supports

```
id

name

↓

id

name

email
```

Does NOT support

```
INTEGER

↓

STRING
```

---

# Common Errors

```
Wrong Delimiter

Wrong Header

Missing Columns

Extra Columns

Wrong Data Types

Corrupted Files

Empty Files
```

---

# Best Practices

✅ Use COPY INTO for batch ingestion.

✅ Keep Bronze append-only.

✅ Use immutable source files.

✅ Keep filenames unique.

✅ Schedule COPY INTO using Databricks Jobs.

✅ Validate files before loading.

✅ Use PATTERN for dynamic file selection.

---

# Common Certification Traps

## Trap 1

COPY INTO overwrites existing data.

❌ False

It appends new files and skips previously processed files.

---

## Trap 2

Metadata is updated before writing.

❌ False

Metadata is updated after a successful commit.

---

## Trap 3

FORCE prevents duplicates.

❌ False

FORCE reloads files and may introduce duplicates.

---

## Trap 4

mergeSchema converts INTEGER into STRING.

❌ False

It only supports adding compatible new columns.

---

## Trap 5

COPY INTO is used for continuous ingestion.

❌ False

Auto Loader is designed for continuous ingestion.

---

# Memory Tricks

## Batch

```
COPY INTO
```

---

## Streaming

```
Auto Loader
```

---

## Processing

```
Read

↓

Write

↓

Commit

↓

Metadata
```

---

## Metadata

```
Loaded Before

↓

Skip
```

---

## FORCE

```
Ignore Metadata

↓

Reload
```

---

# Associate Exam Questions

### Question 1

What is the primary purpose of COPY INTO?

✅ Batch file ingestion.

---

### Question 2

Why is COPY INTO idempotent?

✅ It tracks loaded files using metadata.

---

### Question 3

When is metadata updated?

✅ After a successful commit.

---

### Question 4

What does FORCE = TRUE do?

✅ Reloads previously processed files.

---

### Question 5

Difference between FILES and PATTERN?

✅ FILES specifies exact filenames.

✅ PATTERN uses regular expressions.

---

### Question 6

Which option controls delimiter?

✅ FORMAT_OPTIONS

---

### Question 7

Can mergeSchema change INTEGER into STRING?

❌ No.

---

### Question 8

Should COPY INTO be used for continuous streaming?

❌ No.

---

### Question 9

Why should corrected files use new filenames?

✅ Because COPY INTO identifies processed files using its load metadata and skips previously processed files by default.

---

### Question 10

Where should COPY INTO load data?

✅ Bronze Layer.

---

# Professional Interview Questions

## Q1

Why is COPY INTO idempotent?

**Answer**

Because Databricks stores metadata for previously loaded files. Future executions compare incoming files against this metadata and skip files that have already been processed.

---

## Q2

Why is metadata updated after commit?

**Answer**

Updating metadata before the commit could cause data loss if the write fails. Updating it after the commit guarantees safe retries.

---

## Q3

When would you choose COPY INTO instead of Auto Loader?

**Answer**

For historical migrations, scheduled batch ingestion, and one-time file loads where a continuously running streaming job is unnecessary.

---

## Q4

What happens when a corrected file is uploaded with the same filename?

**Answer**

COPY INTO skips the file because its metadata already exists. Upload the corrected file with a new filename, or intentionally reload it using `FORCE = TRUE` and handle duplicate records downstream.

---

## Q5

Design a nightly batch ingestion pipeline.

**Answer**

- Land daily files in cloud storage.
- Schedule a Databricks Job (or Airflow/ADF) to execute COPY INTO.
- Load raw data into the Bronze layer.
- Apply validation, cleansing, and enrichment in Silver.
- Publish business-ready datasets to Gold.

---

# 5-Minute Revision

Remember these six ideas.

### 1. COPY INTO

```
Batch

↓

Scheduled
```

---

### 2. Lifecycle

```
List

↓

Metadata

↓

Read

↓

Write

↓

Commit

↓

Metadata
```

---

### 3. Idempotency

```
Already Loaded

↓

Skip
```

---

### 4. FORCE

```
Ignore Metadata

↓

Reload
```

---

### 5. FILES

```
Known Files
```

---

### 6. PATTERN

```
Regex

↓

Dynamic Files
```

---

# Final Summary

COPY INTO is Databricks' recommended solution for **batch file ingestion**.

It provides:

- Idempotent loading
- Automatic metadata tracking
- Support for multiple cloud storage platforms
- Flexible file selection
- Reliable and scalable batch ingestion

COPY INTO is ideal for historical migrations and scheduled batch pipelines, while Auto Loader remains the preferred solution for continuous ingestion.