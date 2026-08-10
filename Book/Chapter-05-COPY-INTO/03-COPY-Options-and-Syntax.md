# 03 - COPY INTO Options and Syntax

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- COPY INTO Syntax
- FILEFORMAT
- FILES
- PATTERN
- FORMAT_OPTIONS
- COPY_OPTIONS
- FORCE
- MERGESCHEMA
- Different File Formats
- Common Loading Scenarios
- Best Practices
- Interview Questions

---

# Basic Syntax

The simplest COPY INTO command is:

```sql
COPY INTO bronze.orders
FROM 's3://company-data/orders/'
FILEFORMAT = CSV;
```

Execution Flow

```
Cloud Storage

↓

COPY INTO

↓

Read New Files

↓

Write Delta Table
```

---

# Syntax Structure

```sql
COPY INTO <table_name>
FROM <path>
FILEFORMAT = <format>
FORMAT_OPTIONS (...)
COPY_OPTIONS (...);
```

Not every option is mandatory.

The minimum required options are:

- Target Table
- Source Path
- File Format

---

# FILEFORMAT

Defines the format of incoming files.

Supported formats

| Format | Supported |
|----------|-----------|
| CSV | ✅ |
| JSON | ✅ |
| PARQUET | ✅ |
| AVRO | ✅ |
| ORC | ✅ |
| TEXT | ✅ |

Example

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
FILEFORMAT = PARQUET;
```

---

# FILES Option

Sometimes you don't want to load every file.

Instead, load only specific files.

Example

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
FILES=('jan.csv','feb.csv')
FILEFORMAT=CSV;
```

Only these files are processed.

```
sales/

jan.csv   ✅

feb.csv   ✅

march.csv ❌
```

---

# PATTERN Option

Instead of specifying filenames individually,

you can use regular expressions.

Example

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
PATTERN='.*2026.*'
FILEFORMAT=CSV;
```

Example

```
sales_2025.csv

sales_2026.csv

sales_2027.csv
```

Only

```
sales_2026.csv
```

matches the pattern.

---

# FILES vs PATTERN

| FILES | PATTERN |
|--------|----------|
| Explicit filenames | Regular expression |
| Small file lists | Dynamic file selection |
| Manual | Flexible |

---

# FORMAT_OPTIONS

FORMAT_OPTIONS describe **how the file should be interpreted**.

Example

```sql
FORMAT_OPTIONS(
'header'='true',
'delimiter'=','
)
```

These options do not change the data.

They tell Databricks how to read the file.

---

# Common FORMAT_OPTIONS

## Header

CSV

```
id,name,salary
```

Example

```sql
FORMAT_OPTIONS(
'header'='true'
)
```

Without it

```
id

name

salary
```

would be treated as data.

---

## Delimiter

Default CSV delimiter

```
,
```

Suppose

```
1;Alice;50000
```

You must specify

```sql
FORMAT_OPTIONS(
'delimiter'=';'
)
```

Otherwise the entire row is read as one column.

---

## Quote

Example

```
"John Doe",50000
```

Specify

```sql
FORMAT_OPTIONS(
'quote'='"'
)
```

---

## Escape

Suppose

```
John \"Smith\"
```

Specify

```sql
FORMAT_OPTIONS(
'escape'='\\'
)
```

---

## Null Value

Suppose

```
NULL
```

should become SQL NULL.

```sql
FORMAT_OPTIONS(
'nullValue'='NULL'
)
```

---

## Date Format

```sql
FORMAT_OPTIONS(
'dateFormat'='yyyy-MM-dd'
)
```

---

## Timestamp Format

```sql
FORMAT_OPTIONS(
'timestampFormat'='yyyy-MM-dd HH:mm:ss'
)
```

---

## MultiLine

Useful for JSON or CSV containing embedded newlines.

```sql
FORMAT_OPTIONS(
'multiLine'='true'
)
```

---

# COPY_OPTIONS

COPY_OPTIONS control **how COPY INTO behaves**.

Unlike FORMAT_OPTIONS,

they affect loading behavior rather than file parsing.

---

# FORCE

Default

```sql
FORCE='false'
```

Behavior

```
Metadata Exists

↓

Skip File
```

When

```sql
FORCE='true'
```

Behavior

```
Ignore Metadata

↓

Reload File
```

Use carefully.

---

# MERGESCHEMA

Suppose

Yesterday

```
id

name
```

Today

```
id

name

email
```

Example

```sql
COPY_OPTIONS(
'mergeSchema'='true'
)
```

Result

```
email
```

is automatically added.

---

# CSV Example

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
FILEFORMAT=CSV
FORMAT_OPTIONS(
'header'='true',
'delimiter'=','
);
```

---

# JSON Example

```sql
COPY INTO bronze.orders
FROM 's3://orders/'
FILEFORMAT=JSON;
```

---

# Parquet Example

```sql
COPY INTO bronze.customer
FROM 's3://customer/'
FILEFORMAT=PARQUET;
```

No delimiter or header options are required.

---

# Loading Selected Files

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
FILES(
'jan.csv',
'feb.csv'
)
FILEFORMAT=CSV;
```

---

# Loading Files Using Pattern

```sql
COPY INTO bronze.sales
FROM 's3://sales/'
PATTERN='.*2026.*'
FILEFORMAT=CSV;
```

---

# Real Enterprise Example

Folder

```
sales/

sales_2025.csv

sales_2026.csv

sales_test.csv

backup.csv
```

Requirement

Load only production files for 2026.

Solution

```sql
PATTERN='sales_2026.*'
```

---

# Choosing FILES vs PATTERN

Use **FILES** when:

- A small number of known files must be loaded.
- The filenames are fixed.

Use **PATTERN** when:

- Files follow a naming convention.
- The file list changes over time.
- Dynamic selection is required.

---

# Best Practices

✅ Use PATTERN for dynamic datasets.

✅ Use FILES for one-time loads.

✅ Always specify header correctly for CSV.

✅ Match delimiter with the source file.

✅ Use FORCE only when intentional reprocessing is required.

✅ Test COPY commands on a small sample before production.

---

# Common Mistakes

❌ Forgetting to set delimiter.

❌ Using header=false when a header exists.

❌ Using FORCE unnecessarily.

❌ Incorrect PATTERN expressions.

❌ Expecting mergeSchema to convert data types.

---

# Associate Exam Notes

Remember

```
FILES

↓

Specific Files

----------------

PATTERN

↓

Regex

----------------

FORMAT_OPTIONS

↓

Read File

----------------

COPY_OPTIONS

↓

Loading Behavior

----------------

FORCE

↓

Reload Files

----------------

MERGESCHEMA

↓

Add Columns
```

---

# Professional Interview Questions

## Question 1

When would you choose FILES instead of PATTERN?

**Expected Answer**

Use FILES when loading a known set of files. PATTERN is preferred when files follow a naming convention and dynamic selection is required.

---

## Question 2

Why is delimiter part of FORMAT_OPTIONS instead of COPY_OPTIONS?

**Expected Answer**

Because delimiter controls how the input file is parsed. It affects file interpretation, not the loading behavior.

---

## Question 3

A CSV uses semicolons as delimiters.

What happens if delimiter is not specified?

**Expected Answer**

The parser assumes commas by default, causing the entire row to be interpreted as a single column or resulting in incorrect parsing.

---

## Question 4

Does mergeSchema automatically convert INTEGER columns to STRING?

**Expected Answer**

No. mergeSchema supports adding new columns but does not automatically handle incompatible data type changes.

---

# Chapter Summary

```
COPY INTO

↓

FILEFORMAT

↓

FORMAT_OPTIONS

↓

COPY_OPTIONS

↓

Read Files

↓

Write Delta
```

---

# Key Takeaways

- FILEFORMAT defines the type of input file.
- FILES loads specific files.
- PATTERN loads files matching a regular expression.
- FORMAT_OPTIONS control how files are parsed.
- COPY_OPTIONS control loading behavior.
- FORCE reloads previously processed files.
- mergeSchema automatically adds compatible new columns but does not change incompatible data types.