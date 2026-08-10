# 07 - Cost Optimization & Best Practices

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# 📖 Introduction

Running Databricks workloads efficiently is not just about performance—it is also about controlling infrastructure costs while maintaining reliability and scalability.

Enterprise organizations execute thousands of jobs every day. Poor compute decisions can significantly increase cloud costs, while well-designed architectures improve performance and reduce operational expenses.

This chapter focuses on the best practices used by production teams to optimize Databricks Compute.

---

# 🤔 Why Cost Optimization Matters?

Imagine two organizations.

### Organization A

```
100 Workers

↓

Running 24×7

↓

Only 20% Utilization
```

---

### Organization B

```
Autoscaling

+

Auto Termination

+

Job Compute

+

Photon

+

Instance Pool
```

Question:

Which organization spends less money?

The answer is **Organization B**, because compute resources are allocated only when required.

---

# Understanding Compute Cost

The primary cost in Databricks comes from:

- Cloud Infrastructure (Virtual Machines)
- Databricks Units (DBUs)
- Compute Runtime

Storage is comparatively inexpensive.

Keeping large clusters running without work is one of the biggest sources of unnecessary cloud cost.

---

# Best Practice 1 - Choose the Right Compute

Choosing the appropriate Compute type is the first optimization step.

| Workload | Recommended Compute |
|----------|---------------------|
| Notebook Development | All-Purpose Compute |
| Scheduled ETL | Job Compute |
| BI Dashboards | SQL Warehouse |
| Minimal Infrastructure Management | Serverless Compute |

Using the wrong Compute type increases both cost and operational complexity.

---

# Best Practice 2 - Enable Auto Termination

Development clusters are often forgotten after users finish working.

Example:

```
Developer

↓

Lunch Break

↓

Cluster Idle

↓

Still Running
```

Configure:

```
Auto Termination

↓

20–30 Minutes
```

The cluster automatically stops after the configured idle period, reducing unnecessary cloud costs.

---

# Best Practice 3 - Use Autoscaling

Workloads vary throughout the day.

Example:

Morning

```
100 GB
```

Afternoon

```
10 TB
```

Night

```
50 GB
```

Instead of permanently allocating a large cluster, configure Autoscaling with appropriate minimum and maximum worker limits.

Benefits:

- Better resource utilization
- Lower infrastructure cost
- Improved scalability

---

# Best Practice 4 - Use Instance Pools

Creating new virtual machines for every Job Cluster increases startup time.

Use Instance Pools when:

- Large numbers of short-lived Job Clusters are created.
- CI/CD pipelines frequently launch clusters.
- Startup latency affects SLAs.

Remember:

Instance Pools reduce **cluster startup time**, not Spark execution time.

---

# Best Practice 5 - Enable Photon

Photon accelerates many SQL and DataFrame workloads.

Recommended for:

- Joins
- Aggregations
- MERGE
- OPTIMIZE
- Delta Lake operations

Photon improves execution performance without requiring code changes.

---

# Best Practice 6 - Use LTS Runtime for Production

Long-Term Support (LTS) Runtime provides:

- Stability
- Security updates
- Bug fixes
- Predictable behavior

Do not immediately upgrade production workloads to the latest Runtime version.

Instead:

```
Development

↓

QA

↓

UAT

↓

Production
```

Validate compatibility before deployment.

---

# Best Practice 7 - Right-Size Clusters

Bigger clusters do not always produce faster jobs.

Before increasing cluster size:

- Review execution plans
- Check data skew
- Optimize partitions
- Enable Photon
- Optimize Delta tables

Scale compute only after identifying the actual bottleneck.

---

# Best Practice 8 - Separate Workloads

Avoid using one cluster for every workload.

Recommended architecture:

```
Developers

↓

All-Purpose Compute

---------------------

Nightly ETL

↓

Job Compute

---------------------

Business Intelligence

↓

SQL Warehouse
```

Benefits:

- Better performance
- Improved workload isolation
- Lower cost
- Easier resource management

---

# Best Practice 9 - Monitor Compute Usage

Regularly review:

- Idle clusters
- Autoscaling behavior
- Cluster utilization
- Job duration
- Startup time
- Failed jobs

Monitoring helps identify optimization opportunities before costs increase.

---

# Best Practice 10 - Optimize Before Scaling

A common mistake is increasing cluster size whenever jobs become slow.

Correct approach:

```
Slow Pipeline

↓

Identify Bottleneck

↓

Optimize Spark Code

↓

Optimize Delta Tables

↓

Enable Photon

↓

Tune Partitions

↓

Increase Compute (Only if Required)
```

Always optimize the workload before increasing infrastructure.

---

# Production Architecture

```
                    Databricks Workspace

                            │

          ┌─────────────────┼──────────────────┐

          ▼                 ▼                  ▼

    All-Purpose       Job Compute        SQL Warehouse

     Development          ETL                 BI

                            │

                            ▼

                     Autoscaling

                            │

                     Instance Pool

                            │

                    Photon Enabled

                            │

                     LTS Runtime

                            │

                   Auto Termination

                            │

                      Delta Lake
```

Each component solves a different operational problem.

---

# Common Production Mistakes

## ❌ Running All-Purpose Compute for Production ETL

Use Job Compute instead.

---

## ❌ Leaving Clusters Running Overnight

Configure Auto Termination.

---

## ❌ Using One Cluster for Every Team

Separate development, ETL, and BI workloads.

---

## ❌ Increasing Cluster Size Without Analysis

Optimize the application before scaling infrastructure.

---

## ❌ Upgrading Production Immediately

Always validate Runtime upgrades in lower environments before production deployment.

---

# 🎓 Associate Certification Focus

Remember:

```
Development

↓

All-Purpose

-------------------

ETL

↓

Job Compute

-------------------

Dashboards

↓

SQL Warehouse

-------------------

SQL Performance

↓

Photon

-------------------

Startup Time

↓

Instance Pool

-------------------

Idle Cost

↓

Auto Termination

-------------------

Variable Workload

↓

Autoscaling

-------------------

Production

↓

LTS Runtime
```

---

# 🚨 Associate Exam Traps

❌ Photon reduces cluster startup time.

✅ Instance Pools reduce cluster startup time.

---

❌ Autoscaling terminates clusters.

✅ Auto Termination terminates clusters.

---

❌ Job Compute should be used for interactive notebook development.

✅ All-Purpose Compute is designed for interactive development.

---

# 🎯 Associate Practice Questions

### Question 1

A developer forgets to stop a cluster before leaving for the day.

Which feature minimizes unnecessary cost?

A. Photon

B. Auto Termination

C. Instance Pool

D. Runtime ML

---

### Question 2

A nightly ETL job currently runs on an All-Purpose cluster.

Which recommendation best reduces cost?

A. Increase Driver Memory

B. Replace with Job Compute

C. Use Runtime ML

D. Disable Auto Termination

---

### Question 3

A company launches hundreds of short-lived Job Clusters every day.

Which feature reduces startup latency?

A. Autoscaling

B. Photon

C. Instance Pool

D. SQL Warehouse

---

### Question 4

A cluster becomes slow because data volume has increased significantly.

Which feature automatically adds worker nodes?

A. Auto Termination

B. Instance Pool

C. Autoscaling

D. Runtime ML

---

# 🎯 Professional Certification Focus

Professional certification expects you to:

- Design cost-efficient architectures.
- Select appropriate Compute types.
- Optimize production workloads.
- Balance cost and performance.
- Plan Runtime upgrades safely.
- Identify bottlenecks before scaling infrastructure.

---

# 💼 Professional Scenario

A company has:

- 200 Developers
- 500 Daily ETL Jobs
- 100 BI Dashboards
- Strict cost optimization goals

Design a Databricks Compute architecture.

Include:

- Compute Type
- Autoscaling
- Auto Termination
- Instance Pools
- Photon
- Runtime Strategy

Justify why each decision improves performance, scalability, and cost efficiency.

---

# 📋 Chapter Summary

- Select the appropriate Compute type for each workload.
- Enable Auto Termination to eliminate idle costs.
- Use Autoscaling for variable workloads.
- Use Instance Pools to reduce cluster startup time.
- Enable Photon for supported SQL and DataFrame workloads.
- Use LTS Runtime for production stability.
- Optimize workloads before increasing cluster size.

---

# 🔑 Key Takeaways

✅ Choose the right Compute type.

✅ Optimize before scaling.

✅ Autoscaling manages worker count.

✅ Auto Termination reduces idle costs.

✅ Instance Pools reduce startup time.

✅ Photon improves execution performance.

✅ LTS Runtime provides production stability.

✅ Separate workloads for better scalability and lower cost.