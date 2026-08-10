# 06 - Exam Notes and Interview Guide

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Auto Loader Cheat Sheet

## Definition

Auto Loader is a Databricks feature built on Apache Spark Structured Streaming that automatically discovers and incrementally processes new files from cloud object storage while providing scalable, fault-tolerant, and exactly-once file ingestion.

---

# Auto Loader Architecture

```
Cloud Storage

↓

Auto Loader

↓

Structured Streaming

↓

Checkpoint

↓

Bronze Delta Table
```

---

# Internal Processing Flow

```
New File Arrives

↓

File Discovery

↓

Read File

↓

Write Delta

↓

Commit

↓

Update Metadata

↓

Next File
```

**Important:**

Metadata is updated **only after a successful commit**.

---

# File Discovery Modes

## Directory Listing

```
Cloud Storage

↓

List Files

↓

Compare Metadata

↓

Read New Files
```

### Advantages

- Simple setup
- No cloud messaging required

### Disadvantages

- Higher storage API cost
- Slower with billions of files

---

## File Notification

```
Cloud Storage

↓

Cloud Event

↓

Queue

↓

Auto Loader
```

### Advantages

- Event-driven
- Faster discovery
- Lower cloud API cost
- Best for enterprise workloads

---

# Directory Listing vs File Notification

| Feature | Directory Listing | File Notification |
|----------|-------------------|------------------|
| Discovery | List storage | Event-driven |
| Performance | Moderate | High |
| API Cost | Higher | Lower |
| Configuration | Simple | More complex |
| Best For | Small/Medium workloads | Large-scale production |

---

# Auto Loader vs COPY INTO

| Feature | Auto Loader | COPY INTO |
|----------|-------------|-----------|
| Processing | Continuous | Batch |
| Streaming | Yes | No |
| Checkpoint | Yes | No |
| Best Use Case | Continuous ingestion | Historical or scheduled loads |
| Monitoring | Continuous | One-time execution |

---

# Auto Loader vs Structured Streaming

| Auto Loader | Structured Streaming |
|--------------|----------------------|
| File discovery | Streaming engine |
| Schema inference | Micro-batch execution |
| Schema evolution | Fault tolerance |
| Metadata tracking | Checkpoint management |

Remember:

```
Auto Loader

↓

Uses

↓

Structured Streaming
```

---

# Checkpoint

Checkpoint stores:

- Streaming progress
- Processed files
- Offsets
- Commits
- Query metadata

Checkpoint **does not** store business data.

---

# Checkpoint Directory

```
checkpoint/

├── commits/

├── offsets/

├── sources/

├── state/

└── metadata
```

---

# Exactly-Once Processing

Sequence

```
Read File

↓

Write Delta

↓

Commit

↓

Update Metadata
```

Never

```
Read File

↓

Update Metadata

↓

Write Delta
```

That could lead to data loss.

---

# Idempotency

Idempotency means:

Running the same pipeline multiple times produces the same final result.

Example

```
001.csv

↓

Already Recorded

↓

Skip
```

No duplicate ingestion.

---

# Schema Inference

Automatically detects:

- Column Names
- Data Types
- Nullable Columns

---

# Schema Evolution

Supports:

- Adding new columns

Does **not** automatically support:

- Incompatible data type changes

---

# mergeSchema

Supports

```
salary

↓

salary

email
```

Does **not** support

```
INTEGER

↓

STRING
```

Manual intervention is required.

---

# _rescued_data

Unexpected fields are stored in:

```
_rescued_data
```

instead of failing the pipeline.

---

# cloudFiles.allowOverwrites

Default

```python
False
```

Behavior

```
Same Filename

↓

Skip
```

When

```python
True
```

Behavior

```
Same Filename

↓

Reload
```

Use carefully because it may introduce duplicate records.

---

# Small File Problem

Poor

```
100 GB

↓

1,000,000 Files
```

Better

```
100 GB

↓

100 Files
```

Small files increase:

- Listing overhead
- File open/close overhead
- Metadata operations

---

# Best Practices

✅ Ingest into Bronze only

✅ Keep Bronze append-only

✅ Use File Notification for very large workloads

✅ Store checkpoints in durable cloud storage

✅ Monitor `_rescued_data`

✅ Review schema evolution regularly

✅ Use separate checkpoints for each stream

---

# Common Certification Traps

## Trap 1

Auto Loader replaces Structured Streaming.

❌ False

Auto Loader is built on Structured Streaming.

---

## Trap 2

Checkpoint stores table data.

❌ False

It stores streaming metadata only.

---

## Trap 3

Metadata is updated before writing.

❌ False

Metadata is updated after a successful commit.

---

## Trap 4

mergeSchema changes column data types.

❌ False

It only adds compatible new columns.

---

## Trap 5

_rescued_data indicates pipeline failure.

❌ False

It stores unexpected fields while allowing the pipeline to continue.

---

## Trap 6

allowOverwrites = true prevents duplicates.

❌ False

It allows files with the same name to be processed again and may create duplicates if downstream logic does not handle them.

---

# Memory Tricks

## File Discovery

```
Directory Listing

↓

List

--------------------

File Notification

↓

Listen
```

---

## Storage

```
Checkpoint

↓

Streaming Memory

--------------------

Delta Log

↓

Table Memory
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

## Schema

```
Inference

↓

Detect

--------------------

Evolution

↓

Adapt

--------------------

mergeSchema

↓

Add Columns

--------------------

_rescued_data

↓

Unexpected Fields
```

---

# Associate Exam Questions

### Question 1

Auto Loader is built on which technology?

✅ Apache Spark Structured Streaming

---

### Question 2

Which discovery modes does Auto Loader support?

✅ Directory Listing

✅ File Notification

---

### Question 3

Where is streaming progress stored?

✅ Checkpoint

---

### Question 4

What does mergeSchema do?

✅ Adds new columns

---

### Question 5

Does mergeSchema automatically handle INTEGER → STRING changes?

❌ No

---

### Question 6

Where are unexpected fields stored?

✅ `_rescued_data`

---

### Question 7

What guarantees exactly-once processing?

✅ Commit before metadata update

---

### Question 8

When is Auto Loader preferred over COPY INTO?

✅ Continuous file ingestion

---

### Question 9

Should business transformations occur in Auto Loader?

❌ No

They belong in the Silver layer.

---

### Question 10

Why should checkpoint directories never be shared?

✅ Each streaming query maintains its own state. Sharing checkpoints can corrupt progress and produce unpredictable results.

---

# Professional Interview Questions

## Q1

Why does Auto Loader update metadata after a successful commit?

**Answer**

Updating metadata before a successful commit could mark a file as processed even if the write fails, leading to data loss. Updating metadata only after a successful commit guarantees fault tolerance and exactly-once processing.

---

## Q2

When would you recommend File Notification Mode?

**Answer**

For enterprise workloads processing millions or billions of files where reducing storage listing operations and API costs is important.

---

## Q3

A source system adds a new column called `customer_type`.

How should the pipeline respond?

**Answer**

Enable schema evolution and mergeSchema so the new column is added automatically to the Bronze table. Review downstream Silver and Gold transformations before using the new field.

---

## Q4

The source changes a column from INTEGER to STRING.

Will Auto Loader automatically fix it?

**Answer**

No. This requires manual analysis, possible casting, or a schema redesign depending on business requirements.

---

## Q5

Why shouldn't heavy transformations run inside Auto Loader?

**Answer**

Auto Loader's primary responsibility is reliable ingestion into the Bronze layer. Business logic, deduplication, joins, and aggregations should occur in downstream Silver pipelines to keep ingestion lightweight and resilient.

---

# 5-Minute Revision

Remember these key ideas:

### 1. Architecture

```
Storage

↓

Auto Loader

↓

Bronze
```

---

### 2. Discovery

```
Directory Listing

↓

Lists Files

--------------------

File Notification

↓

Receives Events
```

---

### 3. Recovery

```
Checkpoint

↓

Streaming Memory
```

---

### 4. Processing

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

### 5. Schema

```
Inference

↓

Detect

Evolution

↓

Adapt
```

---

# Final Summary

Auto Loader is the recommended Databricks solution for continuous file ingestion.

It provides:

- Scalable file discovery
- Incremental ingestion
- Fault tolerance
- Exactly-once processing
- Schema inference
- Schema evolution
- Enterprise-grade reliability

Mastering these concepts is essential for building production Lakehouse ingestion pipelines and for success in the Databricks Data Engineer Associate and Professional certification exams.