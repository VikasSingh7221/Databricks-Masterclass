# Chapter 5 - COPY INTO

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- What is COPY INTO?
- Why COPY INTO was introduced
- COPY INTO Architecture
- Internal Working
- Idempotency
- Metadata Tracking
- FORCE = TRUE
- COPY Options
- Performance Optimization
- Best Practices
- Production Use Cases
- Associate Exam Questions
- Professional Interview Questions

---

# Introduction

Data ingestion is the first step in every data engineering pipeline.

Organizations continuously receive files from multiple sources such as:

- ERP Systems
- CRM Systems
- Banking Applications
- APIs
- IoT Devices
- CSV Exports
- JSON Files
- Parquet Files

These files must be loaded into Delta tables before they can be transformed and analyzed.

Databricks provides **COPY INTO** as a simple, scalable, and idempotent SQL command for loading files from cloud storage into Delta tables.

COPY INTO is primarily designed for **batch ingestion**, making it ideal for one-time loads and scheduled batch pipelines.

---

# What is COPY INTO?

COPY INTO is a SQL command that loads data from cloud object storage into a Delta table.

Unlike a simple INSERT statement, COPY INTO tracks which files have already been loaded, preventing duplicate ingestion during subsequent executions.

Supported cloud storage includes:

- Amazon S3
- Azure Data Lake Storage (ADLS)
- Google Cloud Storage (GCS)

Supported file formats include:

- CSV
- JSON
- Parquet
- Avro
- ORC
- Text

---

# Why Was COPY INTO Introduced?

Traditional file ingestion often required custom scripts to:

1. List files
2. Track processed files
3. Skip duplicate files
4. Handle retries

This increased operational complexity.

COPY INTO simplifies this process by automatically maintaining file load metadata and ensuring idempotent file ingestion.

---

# Where Does COPY INTO Fit?

```
                Source Systems

ERP | CRM | APIs | Logs | CSV

              │

              ▼

        Cloud Storage

              │

              ▼

          COPY INTO

              │

              ▼

       Bronze Delta Table

              │

              ▼

      Silver Delta Table

              │

              ▼

       Gold Delta Table

              │

              ▼

      BI / Analytics / ML
```

COPY INTO is typically used to populate the **Bronze Layer**.

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

Update Metadata
```

Notice that COPY INTO checks previously loaded files before ingestion.

---

# Core Features

COPY INTO provides:

- Batch file ingestion
- Idempotent loading
- Automatic file tracking
- Support for multiple file formats
- Schema validation
- Flexible file selection
- Cloud-native storage support

---

# Chapter Contents

This chapter is divided into five sections.

| File | Topics |
|------|--------|
| 01-COPY-INTO-Fundamentals.md | Introduction, Syntax, Basic Usage |
| 02-Architecture-and-Internals.md | Metadata Tracking, Idempotency, FORCE, Internal Execution |
| 03-COPY-Options.md | FILES, PATTERN, FORMAT_OPTIONS, COPY_OPTIONS |
| 04-Performance-and-Best-Practices.md | Performance, Cost Optimization, Production Design |
| 05-Exam-Notes-and-Interview.md | Certification Notes, Interview Questions, Cheat Sheet |

---

# Skills You Will Gain

After completing this chapter, you will be able to:

- Load files into Delta tables using COPY INTO
- Explain how COPY INTO avoids duplicate file ingestion
- Configure COPY options correctly
- Choose between COPY INTO and Auto Loader
- Design batch ingestion pipelines
- Optimize COPY INTO performance
- Troubleshoot common ingestion issues

---

# Prerequisites

Before starting this chapter, you should understand:

- Lakehouse Architecture
- Delta Tables
- Unity Catalog
- Cloud Object Storage
- Basic SQL

---

# When Should You Use COPY INTO?

COPY INTO is best suited for:

- Historical data migration
- One-time data loading
- Scheduled batch pipelines
- Daily or hourly file ingestion
- Backfilling historical datasets

Examples:

- Load yesterday's sales data
- Import monthly finance reports
- Migrate legacy CSV files
- Scheduled ETL jobs

---

# When Should You NOT Use COPY INTO?

COPY INTO is not ideal for:

- Continuously arriving files
- Near real-time ingestion
- Streaming pipelines

For these use cases, Databricks recommends **Auto Loader**.

---

# COPY INTO vs Auto Loader

| COPY INTO | Auto Loader |
|------------|-------------|
| Batch ingestion | Continuous ingestion |
| SQL command | Structured Streaming based |
| Manual execution or scheduling | Automatic continuous monitoring |
| Best for historical or scheduled loads | Best for continuously arriving files |
| No long-running cluster required | Long-running streaming job |

---

# Key Benefits

✅ Simple SQL syntax

✅ Idempotent file loading

✅ Automatic metadata tracking

✅ Multiple file format support

✅ Easy production deployment

✅ Excellent for batch ingestion

---

# Chapter Summary

COPY INTO is Databricks' recommended solution for **batch file ingestion** into Delta Lake.

It automatically tracks loaded files, prevents duplicate ingestion, and simplifies data loading from cloud storage into Delta tables.

It is ideal for historical data migration and scheduled batch pipelines but should not be used for continuously arriving data, where Auto Loader is the preferred solution.

---

# Next

📖 **01-COPY-INTO-Fundamentals.md**

In the next section, we will study:

- COPY INTO Syntax
- Internal Workflow
- Basic Examples
- Supported File Formats
- Enterprise Use Cases
- COPY INTO vs INSERT
- COPY INTO vs LOAD DATA