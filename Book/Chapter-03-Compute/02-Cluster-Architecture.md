# 02 - Cluster Architecture

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

Every workload executed in Databricks runs on a **Cluster**.

Whether you execute:

- Notebook
- SQL Query
- ETL Pipeline
- Machine Learning
- Streaming Job

Everything eventually runs on a Spark Cluster.

Understanding Cluster Architecture is one of the most important topics because it explains how Spark distributes work across multiple machines.

---

# 🤔 Why was Cluster Architecture Introduced?

Imagine processing

```
100 TB
```

of data on a single machine.

Problems:

- Limited CPU
- Limited Memory
- Very Slow
- Single Point of Failure

One computer cannot efficiently process Big Data.

Databricks solves this problem using a **Distributed Cluster**.

---

# 💡 What is a Cluster?

## Definition

> A Cluster is a collection of multiple Virtual Machines working together to process data in parallel.

A Databricks Cluster consists of:

- One Driver Node
- One or More Worker Nodes

---

# 🏗️ Cluster Architecture

```
                Spark Application

                       │

                  Driver Node

                       │

        ┌──────────────┼──────────────┐

        ▼              ▼              ▼

     Worker 1      Worker 2      Worker 3

        │              │              │

     Executor      Executor      Executor

        │              │              │

   Partition 1    Partition 2    Partition 3
```

---

# Driver Node

## Definition

The Driver is the brain of the Spark Application.

It is responsible for:

- Running your notebook
- Creating DAG
- Creating Execution Plan
- Scheduling Tasks
- Assigning Tasks to Workers
- Collecting Results

The Driver **never processes the actual dataset**.

---

## Driver Responsibilities

- Accept notebook execution
- Create Spark Session
- Create Logical Plan
- Optimize Query
- Create DAG
- Divide work into Stages
- Divide Stages into Tasks
- Send Tasks to Workers
- Collect Results

---

# Worker Nodes

Workers perform the actual computation.

Responsibilities:

- Read data
- Process partitions
- Execute transformations
- Return results

Workers perform the heavy lifting.

---

# Executors

Every Worker contains one or more Executors.

An Executor is responsible for:

- Running Tasks
- Reading Partitions
- Processing Data
- Returning Results

```
Worker

↓

Executor

↓

Task

↓

Partition
```

---

# Spark Application

A Spark Application consists of:

- Driver
- Executors
- Spark Context
- User Code

Everything executes inside a Spark Application.

---

# Spark Execution Flow

Suppose you execute:

```python
df.groupBy("country").sum("sales").show()
```

Execution Flow:

```
Notebook

↓

Driver

↓

Logical Plan

↓

Catalyst Optimizer

↓

Physical Plan

↓

DAG

↓

Stages

↓

Tasks

↓

Workers

↓

Executors

↓

Read Partitions

↓

Process Data

↓

Driver

↓

Result
```

---

# DAG (Directed Acyclic Graph)

Spark converts your code into a DAG.

A DAG represents the sequence of operations required to execute the application.

Characteristics:

- Directed
- No Cycles
- Optimized Execution Plan

---

# Jobs

A Job is created whenever Spark encounters an Action.

Examples:

```python
show()

count()

collect()

write()
```

Each Action creates a new Job.

---

# Stages

A Job consists of one or more Stages.

A new Stage is created whenever Spark encounters a **Shuffle Boundary**.

Examples:

- groupBy()
- join()
- distinct()
- repartition()

---

# Tasks

A Stage consists of multiple Tasks.

The number of Tasks generally depends on the number of Partitions being processed.

Example:

```
8 Partitions

↓

8 Tasks
```

---

# Partitions

Spark divides data into Partitions.

Example:

```
100 GB

↓

Partition 1

Partition 2

Partition 3

Partition 4
```

Each partition can be processed independently.

This enables parallel processing.

---

# Fault Tolerance

Suppose:

```
Worker 3

↓

Crash
```

What happens?

The Driver detects the failure.

The failed Task is reassigned to another available Executor.

Spark recomputes only the failed partition rather than restarting the entire application.

---

# Real Production Example

Suppose an ETL pipeline processes

```
10 TB Sales Data
```

Execution:

```
Notebook

↓

Driver

↓

DAG

↓

3 Stages

↓

120 Tasks

↓

20 Worker Nodes

↓

20 Executors

↓

Parallel Processing

↓

Final Result
```

---

# Best Practices

✅ Increase partitions for larger datasets.

✅ Avoid unnecessary shuffles.

✅ Keep Driver lightweight.

✅ Scale Worker Nodes based on workload.

✅ Monitor failed tasks.

---

# Common Mistakes

## ❌ Mistake 1

Thinking Driver processes data.

Reality:

Driver coordinates execution.

Workers process data.

---

## ❌ Mistake 2

Thinking one Worker processes the entire dataset.

Reality:

Each Worker processes only assigned partitions.

---

## ❌ Mistake 3

Thinking one Action creates multiple Jobs.

Reality:

Each Action creates a new Job.

---

# 🎓 Associate Certification Focus

Remember:

Driver

↓

Coordinates

Workers

↓

Process Data

Executor

↓

Runs Tasks

Job

↓

Created by Action

Stage

↓

Created by Shuffle

Task

↓

Processes Partition

---

# 🚨 Associate Exam Traps

❌ Driver reads data.

✅ Workers read data.

---

❌ Stage depends on number of Actions.

✅ Stage depends on Shuffle Boundaries.

---

❌ Tasks depend on number of Workers.

✅ Tasks depend on number of Partitions.

---

# 🎯 Associate Practice Questions

### Question 1

Which component creates the DAG?

A. Worker

B. Executor

C. Driver

D. Delta Lake

---

### Question 2

Which component processes data?

A. Driver

B. Worker

C. Unity Catalog

D. SQL Warehouse

---

### Question 3

A new Stage is created after:

A. filter()

B. select()

C. groupBy()

D. withColumn()

---

### Question 4

A dataset has

```
16 Partitions
```

How many Tasks will be created?

---

# 🎯 Professional Certification Focus

Professional certification expects you to explain:

- Spark execution internals
- Fault tolerance
- Partitioning strategy
- Shuffle optimization
- Cluster sizing
- Task scheduling
- Resource utilization

---

# 💼 Professional Scenario

A Spark job processes

```
20 TB
```

using

```
2 Workers
```

Execution takes

```
5 Hours
```

How would you optimize the architecture?

Consider:

- Partitions
- Workers
- Autoscaling
- Photon
- Data Skew
- Shuffle Reduction

---

# 📋 Chapter Summary

- A Cluster consists of Driver and Worker Nodes.
- Driver coordinates execution.
- Workers process data.
- Executors execute Tasks.
- Jobs are created by Actions.
- Stages are created by Shuffle Boundaries.
- Tasks process Partitions.
- Spark achieves parallelism through partitioning.

---

# 🔑 Key Takeaways

✅ Driver coordinates execution.

✅ Workers process data.

✅ Executors run Tasks.

✅ Actions create Jobs.

✅ Shuffle creates Stages.

✅ Partitions determine Tasks.

✅ Spark is fault tolerant.