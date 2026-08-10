# 04 - Access Modes

> **Difficulty:** ⭐⭐⭐☆☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

A Databricks Cluster can be used by one user or multiple users. However, not every workload requires the same level of isolation or security.

For example:

- A Data Scientist training an ML model may require a dedicated cluster.
- A team of Data Engineers developing ETL pipelines can share a cluster.
- Business users should not have unrestricted access to development clusters.

To solve these problems, Databricks introduced **Access Modes**.

Access Modes determine **who can use a compute resource and how users are isolated from one another**.

---

# 🤔 Why were Access Modes Introduced?

Imagine an organization with:

- 20 Data Engineers
- 15 Data Scientists
- 200 Analysts

If everyone shares the same cluster:

```
Everyone

↓

One Cluster

↓

Problems
```

Problems:

- Security Risks
- Resource Contention
- Lack of Isolation
- Compliance Issues

Databricks introduced Access Modes to provide controlled and secure access to Compute.

---

# 🎯 What is an Access Mode?

## Definition

> **Access Mode determines how users interact with a Databricks Compute resource and the level of isolation between users sharing that compute.**

Remember:

Access Modes control **Compute Access**, not **Data Access**.

---

# Available Access Modes

```
Access Modes

│

├── Shared

├── Single User (Dedicated)

└── No Isolation Shared (Legacy)
```

---

# 1. Shared Access Mode

## Definition

Multiple users share the same cluster.

Example:

```
Shared Cluster

│

├── Alice

├── Bob

├── Charlie

└── David
```

All users execute workloads on the same Compute resource.

---

## Best Use Cases

- Data Engineering Teams
- Collaborative Development
- Analytics Teams
- Shared Notebook Development

---

## Advantages

- Lower Cost
- Better Resource Utilization
- Easy Collaboration

---

## Limitations

- Resources are shared
- Heavy workloads from one user may impact others

---

# Real Example

A team of 10 Data Engineers develops ETL pipelines together.

Instead of creating 10 clusters,

they share one cluster.

---

# 2. Single User (Dedicated)

## Definition

A cluster is assigned to only one user.

```
Cluster

↓

One User
```

No other users can attach notebooks to the cluster.

---

## Best Use Cases

- Machine Learning
- Sensitive Data Processing
- Personal Development
- Long-running Workloads

---

## Advantages

- Maximum Isolation
- Better Performance
- Enhanced Security

---

## Real Example

A Data Scientist trains a deep learning model for 12 hours.

A dedicated cluster ensures no other workload interferes.

---

# 3. No Isolation Shared (Legacy)

This is an older access mode.

Multiple users share a cluster with minimal isolation.

```
Everyone

↓

Same Cluster

↓

Minimal Isolation
```

Databricks recommends using Shared or Single User modes instead for modern deployments.

---

# Comparison Table

| Feature | Shared | Single User | No Isolation Shared |
|----------|---------|-------------|---------------------|
| Users | Many | One | Many |
| Isolation | Good | Excellent | Minimal |
| Cost | Low | Higher | Low |
| Collaboration | Excellent | Limited | Good |
| Recommended | ✅ | ✅ | ❌ Legacy |

---

# Unity Catalog vs Access Modes

Many beginners confuse these concepts.

Remember:

## Unity Catalog

Controls

```
Who can access DATA
```

Examples:

- Catalog
- Schema
- Table
- View
- Volume
- Row Filters
- Column Masks

---

## Access Modes

Controls

```
Who can access COMPUTE
```

Examples:

- Shared
- Single User
- Legacy

---

## Architecture

```
                Unity Catalog

                     │

          Controls Data Access

                     │

────────────────────────────────

               Access Modes

                     │

         Controls Compute Access
```

One secures the **data**.

The other secures the **compute**.

---

# Real Production Architecture

```
                 Databricks Workspace

                        │

        ┌───────────────┼────────────────┐

        ▼               ▼                ▼

 Shared Cluster   Single User      SQL Warehouse

 Data Engineers  Data Scientists   BI Team

                │

                ▼

          Unity Catalog

       (Controls Data Access)
```

---

# Best Practices

✅ Use Shared Access Mode for collaborative engineering teams.

✅ Use Single User for Machine Learning and sensitive workloads.

✅ Avoid No Isolation Shared in new deployments.

✅ Combine Access Modes with Unity Catalog for complete security.

---

# Common Mistakes

## ❌ Mistake 1

Thinking Access Modes secure data.

Reality:

Access Modes secure Compute.

Unity Catalog secures Data.

---

## ❌ Mistake 2

Using Single User for large engineering teams.

Reality:

Shared mode is more cost-effective for collaborative development.

---

## ❌ Mistake 3

Using No Isolation Shared for new environments.

Reality:

It is considered legacy and is generally not recommended.

---

# 🎓 Associate Certification Focus

Remember:

```
Shared

↓

Many Users

--------------------

Single User

↓

One User

--------------------

Unity Catalog

↓

Data Security

--------------------

Access Modes

↓

Compute Security
```

---

# 🚨 Associate Exam Traps

❌ Access Modes manage table permissions.

✅ Unity Catalog manages table permissions.

---

❌ Unity Catalog controls cluster access.

✅ Access Modes control cluster access.

---

❌ Single User is best for all workloads.

✅ Shared is better for collaborative teams.

---

# 🎯 Associate Practice Questions

### Question 1

Which Access Mode is recommended for a team of Data Engineers working on the same ETL pipelines?

A. Single User

B. Shared

C. No Isolation Shared

D. SQL Warehouse

---

### Question 2

Which Access Mode provides the highest level of isolation?

A. Shared

B. SQL Warehouse

C. Single User

D. Legacy

---

### Question 3

Which service controls access to tables and schemas?

A. Shared Access Mode

B. Driver Node

C. Unity Catalog

D. Worker Node

---

### Question 4

Access Modes primarily control:

A. Data Security

B. Compute Access

C. Storage

D. Delta Tables

---

# 🎯 Professional Certification Focus

Professional certification expects you to design secure Compute architectures.

You should be able to explain:

- Why engineering teams use Shared clusters.
- Why Data Scientists prefer Single User clusters.
- How Access Modes integrate with Unity Catalog.
- Why separating Data Security and Compute Security improves governance.

---

# 💼 Professional Scenario

A company has:

- 30 Data Engineers
- 10 Data Scientists
- 250 Analysts

Design a secure Compute architecture.

Include:

- Appropriate Access Modes
- Unity Catalog integration
- Cost optimization
- Workload isolation

---

# 📋 Chapter Summary

- Access Modes secure Compute.
- Shared mode supports multiple users.
- Single User provides dedicated Compute.
- Legacy mode is not recommended for modern deployments.
- Unity Catalog secures Data, while Access Modes secure Compute.

---

# 🔑 Key Takeaways

✅ Shared → Multiple Users

✅ Single User → One User

✅ Legacy → Avoid for new deployments

✅ Unity Catalog → Data Security

✅ Access Modes → Compute Security