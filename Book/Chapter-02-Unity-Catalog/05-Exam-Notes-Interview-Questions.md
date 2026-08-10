# 05 - Exam Notes & Interview Guide

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Unity Catalog Cheat Sheet

## Object Hierarchy

```
Metastore

↓

Catalog

↓

Schema

↓

Table / View / Volume
```

Remember:

```
Metastore

↓

Catalog

↓

Schema

↓

Objects
```

---

# Unity Catalog Components

| Component | Purpose |
|------------|----------|
| Metastore | Top-level governance container |
| Catalog | Organizes business domains |
| Schema | Organizes related database objects |
| Table | Stores structured data |
| Volume | Stores non-tabular files |
| Storage Credential | Authentication to cloud storage |
| External Location | Storage path + authentication |

---

# Managed vs External Objects

## Managed Table

```
Metadata

+

Data

↓

Managed by Databricks
```

DROP TABLE

↓

Deletes

- Metadata
- Data

---

## External Table

```
Metadata

↓

Unity Catalog

---------------

Data

↓

Amazon S3
```

DROP TABLE

↓

Deletes Metadata

Keeps Files

---

## Managed Volume

Stores files managed by Databricks.

DROP VOLUME

↓

Deletes files.

---

## External Volume

Stores files in customer-managed cloud storage.

DROP VOLUME

↓

Deletes metadata only.

---

# Storage Credentials vs External Locations

## Storage Credential

Answers

```
WHO?
```

Represents cloud authentication.

Examples

- AWS IAM Role
- Azure Managed Identity
- GCP Service Account

---

## External Location

Answers

```
WHERE?
```

Represents the cloud storage path.

Example

```
Storage Credential

+

s3://company-data/

↓

External Location
```

---

# RBAC (Role-Based Access Control)

Recommended Architecture

```
Users

↓

Groups

↓

Privileges

↓

Objects
```

Never

```
Users

↓

Privileges
```

---

# Common Privileges

```
USE CATALOG

USE SCHEMA

SELECT

MODIFY

CREATE TABLE

CREATE VIEW

READ FILES

WRITE FILES
```

---

# GRANT

```
GRANT SELECT
ON TABLE sales.orders
TO analyst_group;
```

---

# REVOKE

```
REVOKE SELECT
ON TABLE sales.orders
FROM analyst_group;
```

---

# Security Features

## Row Filter

Purpose

```
Hide Rows
```

Example

```
Department = Sales

↓

Only Sales Rows Returned
```

---

## Column Mask

Purpose

```
Hide Column Values
```

Example

```
Salary

↓

******
```

---

## Dynamic View

Purpose

Different query results for different users.

Examples

- HR
- Sales
- CEO
- Intern

One View

↓

Different Results

---

# Governance Features

## Lineage

Tracks data movement.

```
CSV

↓

Bronze

↓

Silver

↓

Gold

↓

Dashboard
```

Use Cases

- Root Cause Analysis
- Impact Analysis
- Data Discovery

---

## Audit Logs

Tracks

- Who
- What
- When
- Which Object

Used for

- Compliance
- Security
- Investigation

---

# Best Practices

✅ Use Groups instead of assigning permissions directly to users.

✅ Follow the Principle of Least Privilege.

✅ Use Managed Tables for Bronze, Silver, and Gold.

✅ Use External Tables for externally managed cloud storage.

✅ Reuse Storage Credentials whenever possible.

✅ Create separate External Locations for different storage paths.

✅ Use Row Filters for row-level security.

✅ Use Column Masks for sensitive columns.

✅ Use Dynamic Views for complex security rules.

✅ Enable Audit Logs.

✅ Use Lineage for troubleshooting and impact analysis.

---

# Common Certification Traps

## Trap 1

Storage Credential stores data.

❌ False

It stores authentication only.

---

## Trap 2

External Location stores IAM credentials.

❌ False

It references a Storage Credential.

---

## Trap 3

Managed Tables keep data after DROP TABLE.

❌ False

Both metadata and data are deleted.

---

## Trap 4

External Tables delete S3 files after DROP TABLE.

❌ False

Only metadata is deleted.

---

## Trap 5

Volumes are SQL tables.

❌ False

Volumes store non-tabular files.

---

## Trap 6

USE CATALOG alone is sufficient to query a table.

❌ False

A user typically needs:

- USE CATALOG
- USE SCHEMA
- SELECT

---

## Trap 7

Grant permissions directly to users.

❌ False

Grant permissions to Groups.

---

## Trap 8

Row Filters hide columns.

❌ False

They hide rows.

---

## Trap 9

Column Masks remove rows.

❌ False

They hide column values.

---

## Trap 10

Audit Logs track data lineage.

❌ False

Audit Logs track user activity.

Lineage tracks data movement.

---

# Memory Tricks

## Storage

```
Storage Credential

↓

WHO?

--------------

External Location

↓

WHERE?
```

---

## Security

```
Hide Rows

↓

Row Filter

----------------

Hide Values

↓

Column Mask
```

---

## Governance

```
Who accessed data?

↓

Audit Logs

----------------

Where did data come from?

↓

Lineage
```

---

## RBAC

```
Users

↓

Groups

↓

Permissions
```

---

# Associate Exam Questions

## Question 1

Which Unity Catalog object stores cloud authentication?

✅ Storage Credential

---

## Question 2

Which object defines a cloud storage path?

✅ External Location

---

## Question 3

Which object stores images and PDFs?

✅ Volume

---

## Question 4

Which feature hides rows?

✅ Row Filter

---

## Question 5

Which feature hides column values?

✅ Column Mask

---

## Question 6

Which feature tracks data movement?

✅ Lineage

---

## Question 7

Which feature records user activity?

✅ Audit Logs

---

## Question 8

What happens when a Managed Table is dropped?

✅ Metadata and data are deleted.

---

## Question 9

What happens when an External Table is dropped?

✅ Metadata is deleted.

Data files remain.

---

## Question 10

Should permissions be granted to users or groups?

✅ Groups

---

# Professional Interview Questions

## Q1

Why should Storage Credentials and External Locations be separate?

Expected Answer:

Authentication and storage paths are separate responsibilities. One Storage Credential can be reused across multiple External Locations, reducing duplication and simplifying administration.

---

## Q2

Why use Groups instead of individual user permissions?

Expected Answer:

Groups simplify permission management, improve scalability, and reduce administrative overhead.

---

## Q3

When would you use an External Table?

Expected Answer:

When data already exists in cloud storage or must remain externally managed.

---

## Q4

A compliance officer asks:

"Who viewed salary information yesterday?"

Expected Answer:

Audit Logs.

---

## Q5

A dashboard contains incorrect data.

How do you identify the source?

Expected Answer:

Use Lineage to trace the data flow from the dashboard through Gold, Silver, Bronze, and back to the source.

---

## Q6

Should business users query Bronze tables directly?

Expected Answer:

No. Bronze contains raw data. Business users should query curated Gold tables.

---

# 5-Minute Revision

Remember these five ideas:

1. **Hierarchy**

```
Metastore → Catalog → Schema → Table
```

2. **Managed vs External**

- Managed = Databricks manages data.
- External = Customer manages data.

3. **Storage**

- Storage Credential = WHO
- External Location = WHERE

4. **Security**

- Row Filter = Hide Rows
- Column Mask = Hide Values
- Dynamic View = Complex Logic

5. **Governance**

- Audit Logs = Who did what?
- Lineage = Where did data come from?

---

# Final Summary

Unity Catalog provides centralized governance for the Lakehouse.

It enables:

- Secure access control
- Centralized metadata
- Fine-grained permissions
- Data lineage
- Auditing
- Data discovery
- Enterprise-scale governance

Mastering these concepts is essential for both the Databricks Data Engineer Associate and Professional certifications.