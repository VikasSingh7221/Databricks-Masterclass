# 05 - Databricks Runtime & Photon

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

Apache Spark is the distributed processing engine used by Databricks to process large-scale data. However, Databricks does not use plain Apache Spark.

Instead, it provides **Databricks Runtime (DBR)**, an optimized distribution of Apache Spark that includes performance improvements, cloud integrations, Delta Lake support, security enhancements, and additional libraries.

For even higher performance, Databricks provides **Photon**, a native execution engine that accelerates many SQL and DataFrame workloads without requiring code changes.

Understanding Databricks Runtime and Photon is essential for designing high-performance and production-ready data pipelines.

---

# 🤔 Why was Databricks Runtime Introduced?

Apache Spark is designed as a general-purpose distributed processing engine.

It supports:

- On-Premise
- AWS
- Azure
- Google Cloud
- Kubernetes
- Hadoop

Because Spark is designed for multiple environments, it cannot include platform-specific optimizations for every cloud provider.

Databricks introduced its own Runtime to provide:

- Better Performance
- Native Delta Lake Integration
- Built-in Libraries
- Cloud Optimizations
- Security Enhancements
- Easier Cluster Management

Instead of modifying Apache Spark itself, Databricks built an optimized distribution called **Databricks Runtime**.

---

# 💡 What is Databricks Runtime?

## Definition

> **Databricks Runtime (DBR) is an optimized distribution of Apache Spark that includes performance enhancements, Delta Lake integration, cloud optimizations, security improvements, and additional libraries.**

Think of it as:

```
Apache Spark

      +

Delta Lake

      +

Cloud Optimizations

      +

Security

      +

Optimized Libraries

      +

Photon (Optional)

      =

Databricks Runtime
```

---

# Apache Spark vs Databricks Runtime

| Apache Spark | Databricks Runtime |
|---------------|-------------------|
| Open Source | Optimized by Databricks |
| Generic Spark Engine | Spark + Optimizations |
| Manual Library Installation | Pre-installed Libraries |
| Community Managed | Databricks Managed |
| Basic Performance | Enhanced Performance |

---

# Components of Databricks Runtime

Databricks Runtime includes:

- Apache Spark
- Delta Lake
- Cloud Connectors
- Optimized Libraries
- Security Enhancements
- Photon (Optional)

```
Databricks Runtime

│

├── Apache Spark

├── Delta Lake

├── Libraries

├── Cloud Connectors

├── Security

└── Photon
```

---

# Runtime Versions

Databricks regularly releases new Runtime versions.

Examples:

```
13.3 LTS

14.3 LTS

15.x

16.x
```

Every release may include:

- Spark upgrades
- Performance improvements
- Bug fixes
- New Delta Lake features

---

# Long-Term Support (LTS)

LTS stands for:

```
Long-Term Support
```

LTS Runtime is recommended for production because it provides:

- Long support lifecycle
- Security updates
- Bug fixes
- Stable behavior

---

# Latest Runtime

Latest Runtime provides:

- Latest Spark Features
- Latest Databricks Features
- Latest Delta Lake Improvements

However, production environments should validate new Runtime versions before upgrading.

---

# Runtime ML

Machine Learning workloads often require:

- TensorFlow
- PyTorch
- Scikit-learn
- XGBoost

Instead of manually installing these libraries, Databricks provides **Runtime ML**, which includes many commonly used ML libraries.

---

# 🤔 Why was Photon Introduced?

As data volumes increased, SQL queries and DataFrame operations became more computationally expensive.

Databricks wanted to improve execution performance without requiring users to rewrite existing Spark applications.

To solve this problem, Databricks introduced **Photon**.

---

# 💡 What is Photon?

## Definition

> **Photon is Databricks' high-performance native execution engine that accelerates many SQL and DataFrame workloads without requiring code changes.**

Photon is:

- Not a programming language
- Not a storage engine
- Not a replacement for Spark

It is an **execution engine**.

---

# Internal Architecture

Without Photon:

```
Notebook

↓

Spark API

↓

Catalyst Optimizer

↓

Logical Plan

↓

Physical Plan

↓

Spark Execution Engine

↓

CPU

↓

Result
```

With Photon:

```
Notebook

↓

Spark API

↓

Catalyst Optimizer

↓

Logical Plan

↓

Physical Plan

↓

Photon Engine

↓

CPU

↓

Result
```

Notice that:

- Spark APIs remain unchanged.
- Catalyst Optimizer still generates the execution plan.
- Only the execution engine changes.

---

# Why is Photon Faster?

Photon improves performance through:

### Native Execution

Photon is implemented using optimized native code instead of relying solely on the JVM execution path.

---

### Better CPU Utilization

Photon is designed to make better use of modern CPU architectures for supported workloads.

---

### Vectorized Execution

Instead of processing one row at a time, Photon processes batches of rows together.

```
Without Photon

Row

↓

Row

↓

Row

↓

...

-----------------------

With Photon

Batch

↓

Batch

↓

Batch
```

Batch processing reduces execution overhead and improves throughput.

---

# Workloads that Benefit from Photon

Photon accelerates many SQL and DataFrame operations such as:

- Filters
- Joins
- Aggregations
- Sorting
- Delta Reads
- Delta Writes
- MERGE
- OPTIMIZE

---

# Runtime vs Photon

| Databricks Runtime | Photon |
|--------------------|---------|
| Complete Runtime Environment | Execution Engine |
| Includes Spark | Accelerates Execution |
| Includes Libraries | Improves Performance |
| Required for Cluster | Optional Feature |

---

# Real Production Example

A retail company executes a nightly ETL pipeline:

```
Read Delta

↓

Join

↓

Aggregate

↓

MERGE

↓

OPTIMIZE
```

By enabling Photon, the company reduces execution time for supported operations without modifying application code.

---

# Best Practices

✅ Use LTS Runtime for production.

✅ Test new Runtime versions before deployment.

✅ Enable Photon for SQL and DataFrame-heavy workloads.

✅ Use Runtime ML for machine learning projects.

✅ Keep development and production Runtime versions aligned whenever possible.

---

# Common Mistakes

## ❌ Mistake 1

Thinking Databricks Runtime is the same as Apache Spark.

Reality:

Databricks Runtime is an optimized distribution of Apache Spark.

---

## ❌ Mistake 2

Thinking Photon requires code changes.

Reality:

Photon accelerates execution without modifying Spark code.

---

## ❌ Mistake 3

Using the latest Runtime directly in production.

Reality:

Validate new Runtime versions before deployment.

---

## ❌ Mistake 4

Expecting Photon to accelerate every workload equally.

Reality:

Photon primarily benefits supported SQL and DataFrame execution paths.

---

# 🎓 Associate Certification Focus

Remember:

```
Apache Spark

↓

Distributed Processing Engine

------------------------

Databricks Runtime

↓

Optimized Spark Distribution

------------------------

Photon

↓

Execution Engine

------------------------

Runtime ML

↓

Machine Learning Libraries

------------------------

LTS

↓

Production Runtime
```

---

# 🚨 Associate Exam Traps

❌ Photon requires rewriting Spark code.

✅ Photon works without changing application code.

---

❌ Databricks Runtime replaces Spark.

✅ Databricks Runtime includes and optimizes Spark.

---

❌ Latest Runtime should always be used in production.

✅ LTS Runtime is generally preferred for production.

---

# 🎯 Associate Practice Questions

### Question 1

What is Databricks Runtime?

A. A storage engine

B. An optimized distribution of Apache Spark

C. A SQL Warehouse

D. A Delta Table

---

### Question 2

Photon primarily improves:

A. Cluster Startup Time

B. SQL and DataFrame Execution

C. Unity Catalog Security

D. Notebook Collaboration

---

### Question 3

Which Runtime is generally recommended for production?

A. Latest Preview Runtime

B. Runtime ML

C. LTS Runtime

D. Legacy Runtime

---

### Question 4

Which statement about Photon is correct?

A. It requires rewriting Spark applications.

B. It replaces Apache Spark APIs.

C. It accelerates supported workloads without changing application code.

D. It only works with SQL.

---

# 🎯 Professional Certification Focus

Professional certification expects you to understand:

- Runtime selection strategy
- Runtime upgrade planning
- Photon performance trade-offs
- Production rollout strategies
- Compatibility validation
- Benchmarking and regression testing

---

# 💼 Professional Scenario

A company processes 15 TB of data every night using SQL joins, aggregations, and MERGE operations.

Management wants to reduce execution time without modifying application code.

Design an optimization strategy using:

- Databricks Runtime
- Photon
- Runtime Version
- Deployment Strategy
- Validation Process

---

# 📋 Chapter Summary

- Databricks Runtime is an optimized distribution of Apache Spark.
- LTS Runtime is recommended for production.
- Runtime ML includes machine learning libraries.
- Photon is a native execution engine.
- Photon improves SQL and DataFrame performance without changing code.
- New Runtime versions should be validated before production deployment.

---

# 🔑 Key Takeaways

✅ Databricks Runtime = Optimized Apache Spark

✅ Photon = Native Execution Engine

✅ Runtime ML = Machine Learning Environment

✅ LTS = Production Stability

✅ Photon accelerates supported SQL and DataFrame workloads without code changes.