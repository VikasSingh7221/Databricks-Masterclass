# 08 - Associate Certification Notes

> **Difficulty:** ⭐⭐⭐☆☆
>
> **Certification:** Databricks Certified Data Engineer Associate

---

# 🎯 Purpose

This document is a **quick revision guide** for the Databricks Certified Data Engineer Associate exam.

It summarizes the most important Compute concepts, common exam traps, comparison tables, and frequently asked questions.

---

# 1. Compute Fundamentals

## Remember

- Compute processes data.
- Storage stores data.
- Compute and Storage scale independently.
- Multiple Compute clusters can access the same Delta table.
- Compute can be created and terminated independently.

### Exam Trap

❌ Compute stores Delta Tables.

✅ Delta Tables are stored in cloud storage.

---

# 2. Driver vs Worker

## Driver

Responsible for:

- Spark Session
- DAG Creation
- Query Planning
- Task Scheduling
- Collecting Results

Driver **does not process data.**

---

## Worker

Responsible for:

- Reading Data
- Processing Partitions
- Executing Tasks

Workers perform the actual computation.

---

# 3. Executor

Remember

```
Worker

↓

Executor

↓

Task

↓

Partition
```

Executors execute Tasks.

---

# 4. DAG

DAG stands for

```
Directed Acyclic Graph
```

Responsible for:

- Execution Plan
- Stage Creation
- Task Scheduling

Created by the Driver.

---

# 5. Jobs

Actions create Jobs.

Examples

- show()
- collect()
- count()
- write()

One Action → One Job

---

# 6. Stages

Stages are created at **Shuffle Boundaries**.

Examples:

- groupBy()
- join()
- distinct()
- repartition()

Narrow transformations (filter, select, withColumn) do **not** create a new stage.

---

# 7. Tasks

Tasks depend on the number of Partitions.

Example

```
8 Partitions

↓

8 Tasks
```

---

# 8. Compute Types

| Workload | Compute |
|----------|----------|
| Notebook Development | All-Purpose |
| Scheduled ETL | Job Compute |
| BI Dashboards | SQL Warehouse |
| Managed Infrastructure | Serverless |

---

# 9. Access Modes

| Requirement | Access Mode |
|-------------|-------------|
| Engineering Team | Shared |
| Individual User | Single User |
| Legacy | No Isolation Shared |

Remember:

Access Modes secure **Compute**.

Unity Catalog secures **Data**.

---

# 10. Databricks Runtime

Databricks Runtime =

```
Apache Spark

+

Delta Lake

+

Cloud Optimizations

+

Security

+

Libraries
```

---

# 11. LTS Runtime

Recommended for

- Production
- Stability
- Long-term Support
- Security Updates

---

# 12. Runtime ML

Includes pre-installed Machine Learning libraries.

Examples

- TensorFlow
- PyTorch
- Scikit-learn
- XGBoost

---

# 13. Photon

Photon is

```
Native Execution Engine
```

Benefits

- Faster SQL
- Faster DataFrames
- Faster Joins
- Faster Aggregations
- Faster MERGE

No code changes required.

---

# 14. Autoscaling

Purpose

```
Adjust Worker Count
```

Scale Up

```
2

↓

4

↓

6
```

Scale Down

```
6

↓

4

↓

2
```

Driver generally does not autoscale.

---

# 15. Auto Termination

Purpose

```
Stop Idle Cluster
```

Example

```
20 Minutes Idle

↓

Cluster Stops
```

---

# 16. Instance Pools

Purpose

```
Reduce Cluster Startup Time
```

Remember

Instance Pool

≠

Cluster

Instance Pool contains

```
Virtual Machines
```

---

# 17. Most Important Comparison Table

| Requirement | Solution |
|-------------|----------|
| Interactive Development | All-Purpose |
| Scheduled ETL | Job Compute |
| BI Dashboards | SQL Warehouse |
| Startup Slow | Instance Pool |
| Variable Workload | Autoscaling |
| Idle Cluster | Auto Termination |
| SQL Performance | Photon |
| Production Runtime | LTS |

---

# 18. Associate Exam Traps

## Trap 1

❌ Driver processes data.

✅ Workers process data.

---

## Trap 2

❌ Photon reduces startup time.

✅ Instance Pool reduces startup time.

---

## Trap 3

❌ Autoscaling stops clusters.

✅ Auto Termination stops clusters.

---

## Trap 4

❌ Access Modes secure data.

✅ Unity Catalog secures data.

---

## Trap 5

❌ Tasks depend on Workers.

✅ Tasks depend on Partitions.

---

## Trap 6

❌ Stages depend on Actions.

✅ Stages depend on Shuffle Boundaries.

---

# 19. Frequently Asked Associate Questions

### Which Compute for Notebook Development?

✅ All-Purpose

---

### Which Compute for Nightly ETL?

✅ Job Compute

---

### Which Compute for Power BI?

✅ SQL Warehouse

---

### Which feature reduces startup time?

✅ Instance Pool

---

### Which feature improves SQL execution?

✅ Photon

---

### Which feature adjusts worker count?

✅ Autoscaling

---

### Which feature stops idle clusters?

✅ Auto Termination

---

### Which Runtime should Production use?

✅ LTS Runtime

---

### Who creates the DAG?

✅ Driver

---

### Who processes data?

✅ Workers

---

### What creates a Job?

✅ Action

---

### What creates a Stage?

✅ Shuffle

---

### What determines the number of Tasks?

✅ Number of Partitions

---

# 20. Last Minute Revision

Remember these mappings:

```
Development

↓

All-Purpose

--------------------

Production ETL

↓

Job Compute

--------------------

SQL

↓

SQL Warehouse

--------------------

Machine Learning

↓

Runtime ML

--------------------

Production

↓

LTS Runtime

--------------------

Performance

↓

Photon

--------------------

Startup Time

↓

Instance Pool

--------------------

Worker Scaling

↓

Autoscaling

--------------------

Idle Cost

↓

Auto Termination

--------------------

Driver

↓

Coordinates

--------------------

Workers

↓

Process Data

--------------------

Executor

↓

Runs Tasks
```

---

# 🏆 Associate Exam Checklist

Before the exam, ensure you can confidently answer:

- ✅ Driver vs Worker
- ✅ Jobs vs Stages vs Tasks
- ✅ Compute Types
- ✅ Access Modes
- ✅ Runtime vs Photon
- ✅ LTS Runtime
- ✅ Autoscaling
- ✅ Auto Termination
- ✅ Instance Pools
- ✅ Cost Optimization
- ✅ Common Exam Traps

If you can explain each topic without looking at notes, you are well prepared for the Compute section of the Databricks Certified Data Engineer Associate exam.