# 02 - Tables and Volumes

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Managed Tables
- External Tables
- Managed Volumes
- External Volumes
- Internal Architecture
- Managed vs External Objects
- Enterprise Use Cases
- Best Practices
- Associate Exam Questions
- Professional Interview Questions

---

# Why Do We Need Tables?

After organizing data using:

```
Metastore

↓

Catalog

↓

Schema
```

we need a place to actually store data.

That is the role of a **Table**.

A table stores structured data in rows and columns.

Example

```
employees

+---------+-------+--------+
| emp_id  | name  | salary |
+---------+-------+--------+
```

---

# Types of Tables

Unity Catalog supports two table types.

```
Tables

│

├── Managed Table

└── External Table
```

This is one of the most important Associate certification topics.

---

# Managed Table

## Definition

A Managed Table is a table whose **metadata and data are both managed by Databricks**.

Databricks decides:

- Where the data is stored
- How it is managed
- When it is deleted

Think of it as:

```
Databricks

↓

Owns Metadata

+

Owns Data
```

---

# Managed Table Architecture

```
User

↓

Unity Catalog

↓

Managed Table

↓

Managed Storage

↓

Delta Files
```

---

# Example

```
CREATE TABLE employees
(
    id INT,
    name STRING,
    salary DOUBLE
);
```

No storage location is specified.

Databricks automatically stores the data in the managed storage location.

---

# What Happens When You Drop a Managed Table?

Suppose

```
DROP TABLE employees;
```

Result

```
Metadata

Deleted

+

Data Files

Deleted
```

Everything is removed.

---

# Advantages

- Easy to manage
- Automatic storage management
- Automatic cleanup
- Recommended for most Lakehouse workloads

---

# External Table

## Definition

An External Table stores **metadata in Unity Catalog**, but the **data remains in an external cloud storage location**.

Think of it as

```
Unity Catalog

↓

Metadata

----------------

Amazon S3

↓

Actual Data
```

---

# Architecture

```
User

↓

Unity Catalog

↓

External Table

↓

Amazon S3

or

Azure Data Lake

or

Google Cloud Storage
```

---

# Example

```
CREATE TABLE sales_data
LOCATION 's3://company-data/sales/';
```

The table metadata is registered in Unity Catalog.

The files remain in S3.

---

# What Happens When You Drop an External Table?

```
DROP TABLE sales_data;
```

Result

```
Metadata

Deleted

----------------

Files in S3

Remain
```

Only the metadata is removed.

The actual data is untouched.

---

# Managed vs External Table

| Feature | Managed Table | External Table |
|----------|---------------|----------------|
| Metadata | Databricks | Databricks |
| Data Files | Databricks Managed Storage | Customer Cloud Storage |
| DROP TABLE | Deletes metadata + data | Deletes metadata only |
| Storage Location | Managed by Databricks | Specified by user |
| Typical Use | Lakehouse tables | Shared or existing data |

---

# When Should You Use Managed Tables?

Use Managed Tables when:

- Building a new Lakehouse
- Databricks manages storage
- You don't need external access
- Simplicity is preferred

Examples:

- Bronze
- Silver
- Gold

---

# When Should You Use External Tables?

Use External Tables when:

- Data already exists in cloud storage
- Multiple platforms share the same data
- Storage lifecycle is managed outside Databricks
- You don't want DROP TABLE deleting files

Examples:

- Shared S3 buckets
- Legacy Data Lakes
- Multi-platform environments

---

# What is a Volume?

A Volume is a Unity Catalog object used to store **non-tabular files**.

Examples

```
CSV

Images

PDF

JSON

XML

Models

Excel

Text Files
```

Instead of rows and columns,

Volumes store files.

---

# Why Not Use Tables?

Tables store structured data.

Volumes store files.

Examples

```
employees.csv

invoice.pdf

image.png

model.pkl
```

These are not tables.

They belong in a Volume.

---

# Types of Volumes

```
Volumes

│

├── Managed Volume

└── External Volume
```

---

# Managed Volume

A Managed Volume is fully managed by Databricks.

```
Unity Catalog

↓

Managed Volume

↓

Managed Storage
```

Dropping the volume removes both metadata and managed files.

---

# External Volume

An External Volume points to a cloud storage location.

```
Unity Catalog

↓

External Volume

↓

Amazon S3
```

Dropping the volume removes the metadata only.

The files remain in cloud storage.

---

# Managed vs External Volume

| Feature | Managed Volume | External Volume |
|----------|----------------|-----------------|
| Metadata | Databricks | Databricks |
| File Storage | Managed Storage | Customer Cloud Storage |
| Drop Volume | Deletes files | Keeps files |
| Typical Use | Internal project files | Shared cloud storage |

---

# Tables vs Volumes

| Tables | Volumes |
|---------|----------|
| Store rows and columns | Store files |
| SQL queries | File operations |
| Structured data | Unstructured or semi-structured files |
| Used by Delta Lake | Used for file storage |

---

# Enterprise Architecture

```
                 Unity Catalog

                      │

      ┌───────────────┼─────────────────┐

      ▼                                 ▼

   Tables                           Volumes

      │                                 │

Structured Data               Non-tabular Files

      │                                 │

      ▼                                 ▼

Delta Tables                CSV / Images / PDFs
```

---

# Best Practices

✅ Use Managed Tables for Bronze, Silver and Gold.

✅ Use External Tables when data must remain outside Databricks.

✅ Store CSV, JSON, Images and Models in Volumes.

✅ Avoid storing business data inside Volumes if it should be queried with SQL.

---

# Common Mistakes

❌ Assuming Tables and Volumes are the same.

❌ Using Volumes for SQL analytics.

❌ Expecting DROP TABLE on an External Table to delete S3 files.

❌ Creating External Tables without proper governance.

---

# Associate Exam Notes

Remember:

```
Managed Table

↓

Metadata + Data

----------------

External Table

↓

Metadata Only

----------------

Managed Volume

↓

Managed Files

----------------

External Volume

↓

External Files
```

---

# Professional Interview Questions

## Question 1

A company already stores 50 TB of Parquet files in Amazon S3.

Should you create a Managed Table or an External Table?

### Expected Answer

An External Table is preferred because the data already exists in S3 and should continue to be managed there. Unity Catalog manages the metadata while the files remain in cloud storage.

---

## Question 2

Why should Bronze, Silver and Gold usually be Managed Tables?

### Expected Answer

Managed Tables simplify lifecycle management because Databricks manages both metadata and storage. They integrate well with Delta Lake features and reduce operational overhead.

---

## Question 3

When should a Volume be used instead of a Table?

### Expected Answer

Volumes are used for storing files such as images, PDFs, CSVs, ML models and other non-tabular content. Tables are intended for structured data that can be queried using SQL.

---

# Chapter Summary

```
                    Unity Catalog

                          │

        ┌─────────────────┼──────────────────┐

        ▼                                    ▼

     Tables                              Volumes

        │                                    │

 Managed / External               Managed / External

        │                                    │

 Structured Data                 Non-tabular Files
```

---

# Key Takeaways

- Managed Tables store both metadata and data under Databricks management.
- External Tables store metadata in Unity Catalog while keeping data in external cloud storage.
- Managed Volumes store files managed by Databricks.
- External Volumes reference externally managed cloud storage.
- Tables are for structured data.
- Volumes are for non-tabular files.