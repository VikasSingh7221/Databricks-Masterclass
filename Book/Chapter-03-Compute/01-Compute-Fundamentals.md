# 01 - Compute Fundamentals

> **Difficulty:** ⭐⭐☆☆☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

Databricks Compute is the processing layer responsible for executing workloads such as notebooks, SQL queries, ETL pipelines, machine learning, and streaming applications.

Unlike traditional systems where storage and compute are tightly coupled, Databricks separates compute from storage. This enables independent scaling, better resource utilization, improved performance, and significant cost savings.

Every operation performed in Databricks eventually runs on a compute resource.

---

# 🤔 Why was Compute Introduced?

Imagine a traditional database server.

```
+------------------------+
| CPU                    |
| Memory                 |
| Storage                |
+------------------------+
```

Everything resides on the same machine.

Problems:

- Expanding storage requires buying a larger server.
- Expanding compute also requires buying a larger server.
- Compute resources remain idle when not processing data.
- High infrastructure cost.

This architecture does not scale efficiently for modern big data workloads.

---

# 💡 Databricks Solution

Databricks separates:

```
Storage

↓

Cloud Storage

(S3 / ADLS / GCS)

--------------------------

Compute

↓

Clusters

↓

Spark
```

Storage and Compute scale independently.

---

# 🎯 What is Compute?

## Definition

> Compute is the collection of cloud resources (CPU, Memory, and Virtual Machines) used to process data in Databricks.

Compute is responsible for:

- Running notebooks
- Executing SQL queries
- Processing Spark jobs
- Running ETL pipelines
- Training Machine Learning models
- Streaming data

---

# 🏗️ Separation of Compute and Storage

```
             Databricks

        ┌────────┴────────┐

        ▼                 ▼

   Compute Layer     Storage Layer

 (Clusters, Spark)    (Delta Tables)

        │                 │

        ▼                 ▼

   Process Data      Store Data
```

This separation is one of the core principles of the Lakehouse Architecture.

---

# 🎯 Why Separate Compute and Storage?

## 1. Independent Scaling

Need more storage?

Increase storage only.

Need more processing power?

Increase compute only.

No dependency exists between the two.

---

## 2. Cost Optimization

Storage remains available permanently.

Compute runs only when required.

```
Storage

↓

Always Available

----------------------

Compute

↓

Start

↓

Run

↓

Terminate
```

You pay for compute only while it is running.

---

## 3. Better Performance

Large datasets require more compute.

Small datasets require fewer resources.

Databricks allows compute resources to be adjusted according to workload.

---

## 4. Resource Sharing

Multiple compute clusters can access the same data simultaneously.

Example:

```
               Delta Table

                    │

      ┌─────────────┼─────────────┐

      ▼             ▼             ▼

 Development     ETL Jobs      SQL Warehouse
```

The same data can be processed by multiple workloads without duplication.

---

# 🏢 Real-World Example

Suppose an e-commerce company stores:

```
100 TB Sales Data
```

Storage remains in cloud storage.

Morning:

- Business Analysts run SQL dashboards.

Afternoon:

- Data Engineers execute ETL jobs.

Night:

- Data Scientists train Machine Learning models.

Each team creates its own compute resources while accessing the same data.

---

# ✅ Benefits of Compute

- Independent scaling
- Better performance
- Cost optimization
- Flexible resource allocation
- Multiple workloads on the same data
- Easy infrastructure management

---

# ⚠️ Common Mistakes

### ❌ Mistake 1

Thinking Compute stores data.

Reality:

Compute processes data.

Storage stores data.

---

### ❌ Mistake 2

Thinking Compute always runs.

Reality:

Clusters can start and terminate based on workload.

---

### ❌ Mistake 3

Thinking larger clusters always improve performance.

Reality:

Performance depends on workload, data layout, code quality, and cluster configuration.

---

# 🎓 Associate Certification Focus

Remember:

- Compute processes data.
- Storage stores data.
- Compute and Storage are independent.
- Compute can be created and terminated on demand.
- Multiple compute clusters can access the same data.

---

# 🚨 Associate Exam Traps

❌ Compute stores Delta Tables.

✅ Delta Tables are stored in cloud storage.

---

❌ Compute and Storage always scale together.

✅ They scale independently.

---

❌ One compute cluster is required per table.

✅ Multiple compute clusters can access the same table.

---

# 🎯 Associate Practice Questions

### Question 1

What is the primary responsibility of Compute in Databricks?

A. Store Delta Tables

B. Process Data

C. Manage Unity Catalog

D. Store Metadata

---

### Question 2

Why does Databricks separate Compute and Storage?

A. To duplicate data

B. To allow independent scaling and cost optimization

C. To eliminate Spark

D. To replace cloud storage

---

### Question 3

Which statement is correct?

A. Compute permanently stores data.

B. Storage executes notebooks.

C. Multiple compute clusters can access the same data.

D. Storage automatically creates clusters.

---

# 🎯 Professional Certification Focus

Professional certification expects you to understand architectural decisions.

You should be able to explain:

- Why separating Compute and Storage reduces cost.
- How independent scaling improves performance.
- Why multiple compute clusters can safely access the same Delta tables.
- How enterprises isolate workloads using different compute resources.

---

# 💼 Professional Scenario

A company currently runs a traditional on-premises data warehouse where compute and storage are tightly coupled.

Management wants to migrate to Databricks.

How would separating Compute and Storage benefit the company?

Consider:

- Scalability
- Cost
- Performance
- Resource Sharing
- Operational Flexibility

---

# 📋 Chapter Summary

- Compute is responsible for processing data.
- Storage is responsible for storing data.
- Databricks separates Compute and Storage.
- Compute can be created, resized, and terminated independently.
- Multiple workloads can access the same data simultaneously.

---

# 🔑 Key Takeaways

✅ Compute processes data.

✅ Storage stores data.

✅ Compute and Storage scale independently.

✅ Compute reduces cost by running only when required.

✅ Separation of Compute and Storage is a fundamental Lakehouse principle.