# 04 - Schema Inference and Evolution

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- What is Schema Inference?
- Why Schema Inference is required
- cloudFiles.schemaLocation
- Schema Evolution
- mergeSchema
- _rescued_data
- Handling New Columns
- Handling Missing Columns
- Handling Data Type Changes
- Production Best Practices
- Associate Exam Notes
- Professional Interview Questions

---

# Introduction

Imagine an application continuously writes CSV files.

Day 1

```
id,name,salary
```

Day 2

```
id,name,salary,email
```

Day 3

```
id,name,salary,email,department
```

The schema keeps changing.

Question:

Should the ingestion pipeline fail every time a new column appears?

Obviously not.

Auto Loader solves this problem through **Schema Inference** and **Schema Evolution**.

---

# What is Schema Inference?

Schema Inference is the process of automatically identifying the structure of incoming data.

Auto Loader analyzes sample files and determines:

- Column Names
- Data Types
- Nullable Columns

Example

CSV

```
id,name,salary

1,Alice,50000

2,Bob,60000
```

Inferred Schema

```
id          INTEGER

name        STRING

salary      DOUBLE
```

No manual schema definition required.

---

# Why is Schema Inference Needed?

Without Schema Inference

```
New File

↓

Developer

↓

Define Schema

↓

Run Pipeline
```

Every schema change requires manual effort.

With Auto Loader

```
New File

↓

Auto Loader

↓

Infer Schema

↓

Continue Processing
```

Much simpler.

---

# cloudFiles.schemaLocation

Auto Loader stores inferred schemas separately from checkpoints.

Example

```
.option(
"cloudFiles.schemaLocation",
"/schemas/orders"
)
```

This directory contains schema metadata.

It allows Auto Loader to compare:

```
Current Schema

↓

Stored Schema
```

and detect changes.

---

# Why Not Store Schema in Checkpoint?

Checkpoint

↓

Tracks streaming progress.

Schema Location

↓

Tracks schema history.

They have different responsibilities.

---

# Schema Evolution

Schema Evolution allows Auto Loader to adapt when the source schema changes.

Suppose

Day 1

```
id,name,salary
```

Day 2

```
id,name,salary,email
```

Auto Loader detects

```
New Column

↓

Schema Evolution

↓

Update Schema
```

No manual modification required.

---

# mergeSchema

Suppose your Bronze Delta table has

```
id

name

salary
```

A new file arrives

```
id

name

salary

email
```

If

```
mergeSchema = true
```

Result

```
id

name

salary

email
```

The new column is automatically added to the Delta table.

---

# What mergeSchema Can Do

✅ Add new columns

Example

```
salary

↓

salary

email
```

No pipeline failure.

---

# What mergeSchema Cannot Do

Suppose

Day 1

```
salary

INTEGER
```

Day 2

```
salary

"Fifty Thousand"
```

Data Type

```
INTEGER

↓

STRING
```

Auto Loader cannot safely determine how to convert all existing data.

This requires manual intervention.

---

# Handling New Columns

Example

Day 1

```
id,name
```

Day 2

```
id,name,email
```

With

```
mergeSchema = true
```

Result

```
id

name

email
```

Pipeline continues successfully.

---

# Handling Missing Columns

Suppose

Day 1

```
id

name

salary
```

Day 2

```
id

name
```

The missing column becomes

```
NULL
```

for those records.

No schema change is required.

---

# Handling Data Type Changes

Day 1

```
salary

INTEGER
```

Day 2

```
salary

STRING
```

Auto Loader cannot automatically evolve incompatible data types.

Possible strategies:

- Fix the upstream source
- Cast values during processing
- Change destination schema after impact analysis

This decision depends on business requirements.

---

# _rescued_data

Suppose Auto Loader encounters unexpected fields or malformed records.

Instead of failing immediately,

Auto Loader can store them in a special column:

```
_rescued_data
```

Example

Incoming JSON

```
{
"id":1,

"name":"Alice",

"unknown_field":"ABC"
}
```

Result

```
id

name

_rescued_data
```

The unexpected data is preserved for later investigation.

---

# Why is _rescued_data Useful?

Without it

```
Unexpected Data

↓

Pipeline Failure
```

With it

```
Unexpected Data

↓

Store in _rescued_data

↓

Pipeline Continues
```

This improves reliability.

---

# Enterprise Example

Suppose a CRM team adds

```
customer_type
```

to the source system.

Auto Loader detects

```
New Column

↓

Schema Evolution

↓

Bronze Updated

↓

Silver Updated Later
```

No manual ingestion changes required.

---

# Best Practices

✅ Store schema metadata separately using schemaLocation.

✅ Enable schema evolution only when appropriate.

✅ Review schema changes before propagating to downstream layers.

✅ Monitor _rescued_data regularly.

✅ Keep Bronze as close to the source as possible.

---

# Common Mistakes

❌ Assuming mergeSchema changes data types.

❌ Ignoring _rescued_data.

❌ Deleting schemaLocation.

❌ Performing manual schema modifications without understanding downstream impact.

---

# Associate Exam Notes

Remember

```
Schema Inference

↓

Detect Schema

---------------------

Schema Evolution

↓

Handle New Columns

---------------------

mergeSchema

↓

Add Columns

---------------------

_rescued_data

↓

Unexpected Data
```

---

# Professional Interview Questions

## Question 1

A source system adds a new column called **email**.

How should Auto Loader handle it?

### Expected Answer

Enable schema evolution and mergeSchema so the new column is automatically added to the Bronze Delta table without interrupting ingestion.

---

## Question 2

The source changes a column from INTEGER to STRING.

Will mergeSchema solve the problem?

### Expected Answer

No. mergeSchema supports adding new columns but does not automatically handle incompatible data type changes. Manual analysis and schema updates are required.

---

## Question 3

Why is _rescued_data important?

### Expected Answer

It allows unexpected or malformed fields to be preserved instead of failing the pipeline, improving reliability while enabling later investigation.

---

# Chapter Summary

```
Incoming Files

↓

Schema Inference

↓

Compare Stored Schema

↓

Schema Evolution

↓

Write Bronze

↓

Unexpected Data

↓

_rescued_data
```

---

# Key Takeaways

- Schema Inference automatically detects incoming schemas.
- Schema Evolution allows pipelines to adapt to new columns.
- `cloudFiles.schemaLocation` stores schema metadata.
- `mergeSchema` automatically adds new columns.
- `mergeSchema` does not handle incompatible data type changes.
- `_rescued_data` preserves unexpected fields instead of failing the pipeline.
- Review schema changes carefully before propagating them downstream.