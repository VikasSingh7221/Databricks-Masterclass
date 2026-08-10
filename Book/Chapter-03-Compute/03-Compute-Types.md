# 03 - Compute Types

> **Difficulty:** ⭐⭐⭐☆☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

Databricks provides multiple Compute types, each designed for a specific workload.

Choosing the correct Compute type is important because it affects:

- Cost
- Performance
- Scalability
- Resource Utilization
- User Experience

One of the most common Associate certification questions is:

> **"Which Compute type should be used for this workload?"**

---

# 🤔 Why Multiple Compute Types?

Imagine one cluster handles:

- Notebook Development
- ETL Jobs
- BI Dashboards
- Machine Learning

Problems:

- Resource Contention
- Slow Queries
- High Cost
- Poor User Experience

Instead of using one cluster for everything, Databricks provides specialized Compute types.

---

# Available Compute Types

Databricks provides four major Compute types.

```
                     Compute

                        │

      ┌─────────────────┼──────────────────┐

      ▼                 ▼                  ▼

 All-Purpose       Job Compute      SQL Warehouse

                        │

                        ▼

                  Serverless Compute
```

Each type solves a different business problem.

---

# 1. All-Purpose Compute

## Definition

All-Purpose Compute is designed for **interactive development**.

It is mainly used by:

- Data Engineers
- Data Scientists
- Data Analysts

Users attach notebooks to the cluster and execute code interactively.

---

## Best Use Cases

- Notebook Development
- Testing
- Debugging
- Data Exploration
- Learning

---

## Advantages

- Interactive
- Supports notebooks
- Easy debugging
- Suitable for collaborative development

---

## Limitations

- More expensive if left running
- Not ideal for scheduled production jobs

---

# Real Example

```
Developer

↓

Notebook

↓

Run

↓

Modify

↓

Run Again
```

This is interactive development.

Use

```
All-Purpose Compute
```

---

# 2. Job Compute

## Definition

Job Compute is created automatically for scheduled jobs.

Workflow

```
Job Starts

↓

Cluster Starts

↓

Execute Job

↓

Cluster Terminates
```

The cluster exists only for the duration of the job.

---

## Best Use Cases

- ETL Pipelines
- Batch Processing
- Scheduled Jobs
- Production Workloads

---

## Advantages

- Lower Cost
- Automatic Termination
- Production Friendly
- No Idle Compute

---

## Real Example

```
1 AM

↓

Job Starts

↓

ETL Executes

↓

Cluster Terminates
```

---

# 3. SQL Warehouse

## Definition

SQL Warehouse is optimized for SQL workloads.

It is designed for:

- Business Intelligence
- Dashboards
- Reporting
- SQL Analytics

---

## Best Use Cases

- Power BI
- Tableau
- Looker
- SQL Queries
- Dashboards

---

## Advantages

- Optimized for SQL
- High Concurrency
- Fast Dashboard Performance
- Supports BI Tools

---

# Real Example

```
Power BI

↓

SQL Warehouse

↓

Dashboard
```

---

# 4. Serverless Compute

## Definition

Serverless Compute removes infrastructure management.

Databricks manages:

- Cluster Creation
- Scaling
- Maintenance
- Updates

The user focuses only on workloads.

---

## Best Use Cases

- Teams wanting minimal infrastructure management
- Quick development
- SQL workloads (where supported)

---

## Advantages

- No Cluster Management
- Automatic Scaling
- Faster User Experience
- Reduced Operational Overhead

---

# Comparison Table

| Feature | All-Purpose | Job Compute | SQL Warehouse | Serverless |
|----------|-------------|-------------|---------------|------------|
| Notebook Development | ✅ | ❌ | ❌ | ✅ |
| ETL Jobs | ❌ | ✅ | ❌ | ⚠️ Depends on workload |
| SQL Queries | ⚠️ | ❌ | ✅ | ✅ |
| Dashboards | ❌ | ❌ | ✅ | ✅ |
| Interactive | ✅ | ❌ | ❌ | ✅ |
| Scheduled Jobs | ❌ | ✅ | ❌ | ⚠️ |

---

# Decision Flow

```
Notebook Development?

↓

YES

↓

All-Purpose

-----------------------

Scheduled ETL?

↓

YES

↓

Job Compute

-----------------------

Power BI?

↓

YES

↓

SQL Warehouse

-----------------------

Don't Want to Manage Infrastructure?

↓

YES

↓

Serverless
```

---

# Real Production Architecture

```
                     Unity Catalog

                           │

      ┌────────────────────┼────────────────────┐

      ▼                    ▼                    ▼

 All-Purpose          Job Compute         SQL Warehouse

 Developers             ETL Jobs           BI Team

                           │

                           ▼

                     Delta Tables
```

---

# Best Practices

✅ Use All-Purpose for development only.

✅ Use Job Compute for production ETL.

✅ Use SQL Warehouse for BI workloads.

✅ Use Serverless when infrastructure management should be minimized.

✅ Separate development and production compute.

---

# Common Mistakes

## ❌ Mistake 1

Using All-Purpose Compute for nightly ETL jobs.

Use Job Compute instead.

---

## ❌ Mistake 2

Using Job Compute for Power BI dashboards.

Use SQL Warehouse.

---

## ❌ Mistake 3

Running all workloads on one cluster.

Separate workloads for better performance and cost optimization.

---

# 🎓 Associate Certification Focus

Remember:

```
Development

↓

All-Purpose

---------------------

ETL

↓

Job Compute

---------------------

SQL

↓

SQL Warehouse

---------------------

No Infrastructure Management

↓

Serverless
```

---

# 🚨 Associate Exam Traps

❌ Nightly ETL → All-Purpose

✅ Nightly ETL → Job Compute

---

❌ Power BI → Job Compute

✅ Power BI → SQL Warehouse

---

❌ Notebook Development → SQL Warehouse

✅ Notebook Development → All-Purpose

---

# 🎯 Associate Practice Questions

### Question 1

A Data Engineer develops notebooks daily.

Which Compute should be used?

A. SQL Warehouse

B. Job Compute

C. All-Purpose Compute

D. Serverless SQL

---

### Question 2

A production ETL pipeline executes every night.

Which Compute is most appropriate?

A. All-Purpose

B. Job Compute

C. SQL Warehouse

D. Dedicated Compute

---

### Question 3

Power BI connects to Databricks.

Which Compute type is recommended?

A. Job Compute

B. SQL Warehouse

C. All-Purpose

D. Runtime ML

---

### Question 4

A startup wants Databricks to manage cluster provisioning and scaling.

Which Compute type best fits this requirement?

A. Job Compute

B. All-Purpose

C. Serverless Compute

D. SQL Warehouse

---

# 🎯 Professional Certification Focus

Professional certification expects you to design architectures using the correct Compute type.

You should be able to:

- Separate development and production workloads.
- Reduce costs by selecting the right Compute.
- Improve concurrency for BI workloads.
- Design enterprise Compute architectures.

---

# 💼 Professional Scenario

A company currently uses one All-Purpose cluster for:

- Notebook Development
- Nightly ETL
- Power BI Dashboards

Problems:

- Slow ETL
- Dashboard delays
- High Cost

Design a new architecture using appropriate Compute types.

---

# 📋 Chapter Summary

- All-Purpose Compute is for interactive development.
- Job Compute is for scheduled production jobs.
- SQL Warehouse is optimized for SQL analytics and BI.
- Serverless Compute reduces infrastructure management.
- Choosing the correct Compute improves cost and performance.

---

# 🔑 Key Takeaways

✅ All-Purpose → Development

✅ Job Compute → ETL

✅ SQL Warehouse → BI & SQL

✅ Serverless → Managed Infrastructure

✅ Separate workloads for better cost, scalability, and reliability.