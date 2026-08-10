# 04 - Error Handling and Validation

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Common COPY INTO failures
- Schema Validation
- Missing Columns
- Extra Columns
- Data Type Mismatches
- Corrupted Files
- Validation Mode
- ON_ERROR
- Production Recovery Strategies
- Best Practices
- Interview Questions

---

# Introduction

Real-world data is rarely perfect.

Files arriving from upstream systems may contain:

- Missing columns
- Extra columns
- Incorrect data types
- Corrupted records
- Wrong delimiters
- Invalid dates
- Empty files

A production ingestion pipeline should be able to detect, report, and recover from these issues.

---

# Common Failure Scenarios

```
Incoming Files

│

├── Missing Columns

├── Extra Columns

├── Wrong Data Types

├── Corrupted Records

├── Invalid Delimiter

├── Invalid Date Format

└── Empty Files
```

Each issue requires a different handling strategy.

---

# Schema Mismatch

Suppose the Delta table has the following schema:

```
id

name

salary
```

Incoming CSV

```
id,name,email
```

Result

```
Schema Mismatch
```

Possible solutions:

- Correct the source schema.
- Enable schema evolution when appropriate.
- Create a mapping layer before loading.

---

# Missing Columns

Destination Table

```
id

name

salary
```

Incoming File

```
id,name
```

The missing column is populated with

```
NULL
```

if allowed by the schema.

Example

| id | name | salary |
|----|------|--------|
| 1 | Alice | NULL |

---

# Extra Columns

Destination Table

```
id

name
```

Incoming File

```
id

name

email
```

Options

- Enable schema evolution if the new column should be stored.
- Ignore the extra column.
- Reject the file based on business requirements.

---

# Data Type Mismatch

Destination

```
salary

INTEGER
```

Incoming

```
salary

"Fifty Thousand"
```

COPY INTO cannot automatically convert incompatible values.

Possible solutions:

- Fix the source data.
- Cast values before loading.
- Redesign the destination schema if appropriate.

---

# Incorrect Delimiter

Suppose the source file uses

```
;
```

instead of

```
,
```

Incorrect configuration

```sql
FORMAT_OPTIONS(
'delimiter'=','
)
```

Result

```
Entire Row

↓

Single Column
```

Correct configuration

```sql
FORMAT_OPTIONS(
'delimiter'=';'
)
```

---

# Header Problems

Suppose the file contains

```
id,name,salary
```

If

```sql
'header'='false'
```

The header is treated as data.

Correct

```sql
FORMAT_OPTIONS(
'header'='true'
)
```

---

# Invalid Date Format

Incoming

```
15/08/2026
```

Expected

```
yyyy-MM-dd
```

Parsing fails.

Solution

```sql
FORMAT_OPTIONS(
'dateFormat'='dd/MM/yyyy'
)
```

---

# Corrupted Records

Examples

```
1,Alice,50000

2,Bob

3,Charlie,70000,ExtraField
```

The second and third rows do not match the expected schema.

Possible strategies:

- Reject the file.
- Correct upstream data.
- Investigate before reprocessing.

---

# Empty Files

Suppose

```
orders.csv
```

contains

```
0 Records
```

COPY INTO completes successfully.

No rows are inserted.

Production pipelines should monitor unexpected empty files.

---

# Validation Mode

Validation allows you to verify files before ingestion.

Typical use cases:

- Testing new datasets
- Checking schema compatibility
- Validating production deliveries

Validation helps identify issues before data reaches the Bronze table.

---

# ON_ERROR

Different organizations follow different error-handling strategies.

Examples include:

- Stop immediately when an error occurs.
- Skip bad records.
- Continue loading while logging failures.

The chosen strategy depends on business and compliance requirements.

---

# Recovery Strategy

```
Failure

↓

Identify Root Cause

↓

Correct Source Data

↓

Reload File

↓

Validate Results
```

Always investigate the root cause before forcing a reload.

---

# Production Example

Suppose a supplier accidentally changes

```
salary

INTEGER
```

to

```
salary

STRING
```

Recommended approach:

1. Pause ingestion if required.
2. Investigate the change.
3. Determine downstream impact.
4. Update schema or transformation logic.
5. Resume processing.

Avoid making schema changes without understanding downstream consumers.

---

# Data Quality Checklist

Before loading:

- Correct delimiter
- Header present
- Expected schema
- Correct data types
- No unexpected columns
- Valid dates
- File not empty

---

# Best Practices

✅ Validate sample files before production.

✅ Monitor failed COPY INTO operations.

✅ Keep Bronze data as close to the source as possible.

✅ Handle schema changes deliberately.

✅ Review upstream changes with source system owners.

---

# Common Mistakes

❌ Assuming all source files follow the same schema.

❌ Ignoring delimiter differences.

❌ Reloading files without investigating failures.

❌ Changing production schemas without impact analysis.

❌ Treating validation as optional.

---

# Associate Exam Notes

Remember

```
Missing Columns

↓

NULL

-------------------

Extra Columns

↓

Schema Evolution (if enabled)

-------------------

Wrong Delimiter

↓

FORMAT_OPTIONS

-------------------

Wrong Data Type

↓

Manual Handling

-------------------

Validation

↓

Check Before Load
```

---

# Professional Interview Questions

## Question 1

A source system starts sending an additional column called `department`.

How should you handle it?

**Expected Answer**

Evaluate whether the new column is required. If yes, enable schema evolution or update the destination schema. Validate downstream impacts before propagating the change.

---

## Question 2

A CSV suddenly uses semicolons instead of commas.

What happens?

**Expected Answer**

If the delimiter is not updated in `FORMAT_OPTIONS`, the parser may treat the entire row as a single column, leading to incorrect ingestion or parsing errors.

---

## Question 3

A column changes from INTEGER to STRING.

Will COPY INTO automatically resolve it?

**Expected Answer**

No. Incompatible data type changes require manual investigation and appropriate schema or transformation updates.

---

## Question 4

Why should validation be performed before loading production data?

**Expected Answer**

Validation identifies schema mismatches, format issues, and data quality problems before data reaches production tables, reducing failures and simplifying recovery.

---

# Chapter Summary

```
Incoming Files

↓

Validation

↓

Schema Check

↓

Read File

↓

Write Delta

↓

Success / Failure
```

Reliable ingestion depends on validating data before loading and handling errors consistently.

---

# Key Takeaways

- Production data often contains quality issues.
- Validate files before loading whenever possible.
- Configure `FORMAT_OPTIONS` correctly.
- Handle schema changes carefully.
- Investigate failures before reprocessing data.
- Design recovery strategies for production pipelines.