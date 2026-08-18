# 09. Compute, Concurrency & Cost

> **Databricks Data Engineer Professional — Certification Notes**
>
> **Source of truth:** Current official Databricks documentation.
>
> **Core principle:**
>
> ```text
> Correctness
>     ↓
> Performance
>     ↓
> Cost
>     ↓
> Operational safety
> ```
>
> Do not optimize one dimension blindly at the expense of the others.

---

# 1. Why Compute Matters

Every Job task needs compute to execute its logic.

Current Lakeflow Jobs supports different compute options, including:

```text
Serverless compute
Classic jobs compute
Classic all-purpose compute
SQL warehouses
Lakeflow pipeline compute
```

The supported/recommended option depends on the task type and workload.

Official documentation:

https://docs.databricks.com/aws/en/jobs/compute

---

# 2. Current Databricks Recommendation

Databricks currently recommends:

> **Use serverless compute for most supported Job workloads.**

Serverless removes the need to manually configure and manage the underlying compute infrastructure.

Conceptually:

```text
Job
 ↓
Serverless
 ↓
Databricks manages infrastructure
 ↓
Task executes
```

---

# 3. Serverless Compute

Serverless compute means Databricks manages the underlying infrastructure.

You generally do not need to manually configure:

```text
Worker count
Driver type
Autoscaling configuration
Cluster startup
Cluster termination
```

for supported serverless workloads.

### Benefits

```text
Less infrastructure management
Automatic provisioning
Automatic scaling
Simpler configuration
Potentially faster operational setup
```

---

# 4. Serverless Is Not Universal

Do not memorize:

```text
Serverless = supports everything
```

This is incorrect.

Current Databricks documentation lists workload-specific limitations.

Examples:

```text
JAR
Spark Submit
Certain streaming configurations
Other unsupported features
```

may require classic compute.

### Professional rule

> **Choose serverless when supported and appropriate; otherwise use the supported classic compute option.**

---

# 5. Compute Selection

Current documentation provides recommendations by task type.

| Task | Recommended compute |
|---|---|
| Notebook | Serverless Jobs |
| Python script | Serverless Jobs |
| Python wheel | Serverless Jobs |
| SQL | Serverless SQL warehouse |
| Lakeflow pipeline | Serverless pipeline |
| dbt | Serverless SQL warehouse |
| dbt CLI | Serverless Jobs |
| JAR | Classic Jobs compute |
| Spark Submit | Classic Jobs compute |

Always verify the current documentation because supported task types and recommendations can change.

---

# 6. Classic Jobs Compute

Classic Jobs compute is manually configured.

You define compute characteristics such as:

```text
Runtime
Instance type
Driver
Workers
Autoscaling
Access mode
Libraries
Spark configuration
```

Use classic compute when:

```text
Serverless does not support the workload
```

or when a specific workload requires capabilities unavailable on serverless.

---

# 7. Jobs Compute vs All-Purpose Compute

For production Jobs, prefer:

```text
Jobs compute
```

rather than:

```text
All-purpose compute
```

Databricks explicitly recommends against using all-purpose compute for production Jobs.

Reasons include:

```text
Different billing
Shared interactive workloads
Less precise cost attribution
Operational differences
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/run-classic-jobs

---

# 8. Dedicated Job Compute

For classic compute:

```text
Job
 ↓
Job compute
 ↓
Task
```

The compute is intended for the automated workload.

This is preferable to running production workloads on an interactive cluster shared by multiple users.

---

# 9. Shared Job Compute

Multiple tasks in the same Job can use the same Job compute resource.

Example:

```text
Job
 │
 ├── Task A
 │
 ├── Task B
 │
 └── Task C
```

All can use:

```text
Shared Job Compute
```

---

# 10. Why Share Compute?

Sharing compute can reduce startup overhead.

Without sharing:

```text
Task A → Cluster A
Task B → Cluster B
Task C → Cluster C
```

With sharing:

```text
Task A ─┐
Task B ─┼→ Shared Job Compute
Task C ─┘
```

This can reduce latency associated with repeatedly starting compute.

Official documentation:

https://docs.databricks.com/aws/en/jobs/compute

---

# 11. Shared Compute Trade-off

Sharing compute is not automatically optimal.

Suppose:

```text
Task A = CPU-heavy
Task B = memory-heavy
Task C = I/O-heavy
```

A single configuration may not be ideal for all workloads.

You may instead use:

```text
Compute A → Task A

Compute B → Task B/C
```

### Professional principle

> **Share compute when workloads are compatible; separate compute when workload characteristics require different optimization.**

---

# 12. Compute Reuse and Startup Latency

If tasks use separate compute:

```text
Task A
 ↓
Start compute
 ↓
Run
 ↓
Task B
 ↓
Start compute
 ↓
Run
```

This can add startup overhead.

Shared compute can reduce this overhead:

```text
Start compute
 ↓
Task A
 ↓
Task B
 ↓
Task C
```

---

# 13. Serverless Performance Modes

Current Databricks serverless Jobs support performance modes.

### Performance-optimized

Designed for:

```text
Faster startup
Fast execution
Latency-sensitive workloads
```

This is currently the default performance mode.

---

### Standard

Designed for:

```text
Automated batch jobs
Workloads that can tolerate longer startup
Cost optimization
```

Current Databricks documentation states Standard mode can reduce costs by **up to 70%** compared with Performance-optimized mode for applicable workloads.

Official documentation:

https://docs.databricks.com/aws/en/compute/serverless/best-practices

---

# 14. Performance Mode Decision

Suppose:

```text
Daily batch
Runs at 2 AM
Startup latency is not critical
```

Consider:

```text
Standard mode
```

Suppose:

```text
Job is highly latency-sensitive
Fast startup is important
```

Consider:

```text
Performance-optimized mode
```

### Important

Do not interpret:

```text
Standard = slow
Performance optimized = always worth the cost
```

Instead:

```text
Business requirement
+
Benchmark
+
Cost
```

should drive the decision.

---

# 15. Compute Sizing

For classic compute, sizing depends on:

```text
Data volume
CPU requirements
Memory requirements
I/O
Shuffle
Parallelism
SLA
Workload type
```

Example:

```text
Small transformation
→ Smaller compute

Large shuffle-heavy transformation
→ More appropriate memory/CPU resources
```

---

# 16. Over-Provisioning

Suppose:

```text
Task uses 20% CPU
Task uses 30% memory
```

but the cluster is significantly oversized.

Potential result:

```text
Higher cost
Little performance improvement
```

Therefore:

> **Measure resource utilization before scaling up.**

---

# 17. Under-Provisioning

Suppose:

```text
CPU = consistently saturated
Memory = near limit
```

and:

```text
Job duration > SLA
```

Then increasing compute capacity may be appropriate.

But first determine the actual bottleneck.

---

# 18. Autoscaling

For classic compute workloads with variable demand, autoscaling can adjust the number of workers based on workload needs.

Conceptually:

```text
Low workload
 ↓
Fewer workers

High workload
 ↓
More workers
```

This can improve resource efficiency for variable workloads.

---

# 19. Autoscaling Is Not a Guarantee of Lower Cost

Autoscaling can reduce unnecessary workers, but:

```text
More workload
→ More workers
→ More compute consumption
```

Therefore monitor:

```text
Runtime
+
Worker count
+
DBUs
+
Cost
```

---

# 20. Concurrency — Two Different Meanings

This is one of the most important Professional concepts.

There are different levels of concurrency:

```text
Job Run concurrency
For Each iteration concurrency
Workspace task-run concurrency
```

Do not mix them.

---

# 21. Job Run Concurrency

Job concurrency controls:

> **How many runs of the same Job can execute at the same time?**

Example:

```text
Max concurrent runs = 1
```

means:

```text
Run 1 → active
Run 2 → cannot execute concurrently
```

Current Databricks default:

```text
Maximum concurrent runs = 1
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/configure-job

---

# 22. Increasing Job Concurrency

Suppose:

```text
Job runs every 15 minutes
Runtime = 45 minutes
```

With:

```text
Max concurrent runs = 1
```

overlapping runs are not allowed.

If the workload is independent and safe to overlap:

```text
Max concurrent runs > 1
```

may be appropriate.

---

# 23. When Multiple Job Runs Make Sense

Examples:

```text
Different processing dates
Independent input parameters
Independent partitions
Independent workloads
```

Example:

```text
Run 1 → processing_date = Aug 16
Run 2 → processing_date = Aug 17
```

If the design is safe:

```text
Both can potentially execute concurrently.
```

---

# 24. When NOT to Increase Job Concurrency

Do not increase concurrency simply because:

> "The Job is waiting."

First ask:

```text
Can runs safely overlap?
```

Potential risks:

```text
Duplicate writes
Race conditions
Table contention
Resource contention
Incorrect ordering
Target-system overload
```

---

# 25. Queueing

Current Databricks Jobs support queueing.

When queueing is enabled:

```text
Run requested
 ↓
Capacity unavailable
 ↓
Run queued
 ↓
Capacity becomes available
 ↓
Run starts
```

Current documentation states queued runs can remain queued for up to:

```text
48 hours
```

if the relevant capacity limitation persists.

---

# 26. Queueing vs Skipping

Without queueing:

```text
Capacity unavailable
      ↓
Run may be skipped
```

With queueing:

```text
Capacity unavailable
      ↓
Run waits
      ↓
Capacity available
      ↓
Run executes
```

### Professional decision

Ask:

> **Is every scheduled/event-driven run required, or can missed runs be skipped?**

---

# 27. Queueing Default

Current Databricks documentation states:

> Queueing is enabled by default for Jobs created through the UI after April 15, 2024.

Do not assume this applies identically to every legacy/API-created Job configuration.

For exam questions, inspect the stated configuration.

---

# 28. Workspace Concurrency

Job concurrency is not the only limit.

Current Databricks documentation states:

```text
Workspace limit = 2000 concurrent task runs
```

If a request cannot start immediately because of workspace-level limits, the run can be affected by queueing/capacity behavior.

---

# 29. For Each Concurrency

For Each has its own concurrency concept.

Example:

```text
For Each
Inputs = 500
Concurrency = 50
```

means:

```text
Up to 50 iterations
```

can execute concurrently.

This is different from:

```text
Max concurrent Job Runs = 50
```

---

# 30. Three Levels of Parallelism

Think:

```text
Workspace
   ↓
Job Runs
   ↓
For Each Iterations
```

Example:

```text
Workspace allows many task runs

Job:
Max concurrent runs = 3

For Each:
Concurrency = 50
```

Potentially:

```text
3 Job Runs
×
50 For Each iterations
```

can create significant task-level parallelism.

This must be evaluated against workspace/source/target capacity.

---

# 31. Concurrency Multiplier

Suppose:

```text
Job concurrency = 5
For Each concurrency = 50
```

Conceptually, you could have:

```text
5 Job Runs
×
50 iterations
=
250 concurrent iteration tasks
```

This is not automatically safe.

You must consider:

```text
Workspace limits
Compute capacity
Source limits
Target limits
Cost
```

---

# 32. Concurrency and Target Bottlenecks

Suppose:

```text
For Each concurrency = 100
```

but the target database handles:

```text
20 concurrent writes
```

Increasing concurrency may cause:

```text
Contention
Throttling
Timeouts
Retries
Longer runtime
```

Therefore:

> **The bottleneck determines useful concurrency.**

---

# 33. Concurrency and Source Bottlenecks

Similarly:

```text
100 concurrent readers
```

may overload the source system.

Possible result:

```text
Source throttling
API rate limits
Connection failures
Slower extraction
```

Therefore evaluate both:

```text
Source capacity
+
Target capacity
```

---

# 34. Concurrency and SLA

Suppose:

```text
SLA = 2 hours
```

Tests show:

```text
Concurrency 10 → 3h
Concurrency 25 → 2h 10m
Concurrency 50 → 1h 40m
Concurrency 100 → 1h 35m
```

A reasonable choice may be:

```text
Concurrency = 50
```

if:

```text
Cost is acceptable
Resource utilization is healthy
50 meets SLA
100 provides little additional benefit
```

### Professional principle

> **Choose the smallest tested concurrency that reliably meets the requirement.**

This is a design principle, not a fixed Databricks product rule.

---

# 35. Performance Optimization

When a Job is slow, don't immediately:

```text
Increase compute
Increase concurrency
```

Use a systematic approach.

```text
1. Identify slow task
2. Inspect timeline
3. Check data volume
4. Check skew
5. Check source
6. Check target
7. Check compute utilization
8. Test configuration
9. Measure
10. Compare cost
```

---

# 36. Critical Path

The critical path is the sequence of dependent work that determines overall completion time.

Example:

```text
A ──→ B ──→ C
      │
      └──→ D
```

If:

```text
A = 10 min
B = 60 min
C = 20 min
D = 10 min
```

then the main path:

```text
A → B → C
```

dominates the Job duration.

Optimizing an unrelated 2-minute task may have almost no effect.

---

# 37. Parallelism

Suppose:

```text
A
├── B
├── C
└── D
```

B/C/D are independent.

They can potentially run in parallel.

```text
A
 ↓
┌───────┬───────┬───────┐
B       C       D
```

This can reduce total execution time.

But parallelism must respect:

```text
Dependencies
Compute
Source
Target
Concurrency
Cost
```

---

# 38. Sequential vs Parallel

### Sequential

```text
A → B → C → D
```

Runtime approximately:

```text
A + B + C + D
```

---

### Parallel

```text
A
├── B
├── C
└── D
```

After A:

```text
max(B, C, D)
```

approximately determines that branch's duration.

### Professional principle

> **Parallelize independent work when the underlying resources can support it.**

---

# 39. Cost Optimization

Cost optimization should not mean:

```text
Always use the cheapest compute
```

Instead:

```text
Optimize cost for the required SLA and correctness.
```

Example:

```text
Option A
Cost = $5
Runtime = 5h

Option B
Cost = $8
Runtime = 1.5h
```

If:

```text
SLA = 2h
```

Option A is not acceptable even though it is cheaper.

---

# 40. Cost vs Performance

Think in terms of:

```text
Cost
  ↕
Performance
```

The goal is usually:

```text
Lowest cost
that satisfies
correctness + SLA + reliability
```

---

# 41. Serverless Cost Optimization

Current Databricks documentation recommends benchmarking a representative workload and analyzing billing data rather than estimating cost purely from configuration.

For serverless usage, cost can be analyzed using:

```text
system.billing.usage
```

Relevant metadata can include:

```text
job_run_id
job_name
```

Official documentation:

https://docs.databricks.com/aws/en/compute/serverless

---

# 42. Cost Attribution

Precise Job cost attribution is easier when using:

```text
Dedicated Job compute
```

or:

```text
Serverless compute
```

because the usage metadata can associate usage with:

```text
job_id
job_run_id
```

---

# 43. All-Purpose Compute Cost Attribution

When a Job runs on all-purpose compute:

```text
Multiple workloads
        ↓
Same cluster
```

The resources are shared.

Therefore precise one-to-one Job cost attribution is difficult.

Current Databricks documentation recommends dedicated Job compute or serverless for accurate Job cost tracking.

---

# 44. DBUs

A **DBU (Databricks Unit)** is a normalized unit of processing power used for Databricks measurement and pricing.

Conceptually:

```text
Compute usage
      ↓
DBUs
      ↓
Billing
```

Do not equate:

```text
DBU = dollar
```

A DBU is a usage unit; pricing depends on the relevant SKU/rate and cloud/service configuration.

---

# 45. Cost Monitoring

Monitor:

```text
Job
Run
Compute
DBUs
Duration
```

Then compare:

```text
Cost per successful run
Cost per million records
Cost per data volume
Cost per business unit
```

These metrics can reveal inefficient workloads.

---

# 46. Cost per Run

Suppose:

```text
Run A = $2
Run B = $2.10
Run C = $4.50
```

Run C deserves investigation.

Potential reasons:

```text
Data growth
Retry
Longer runtime
Compute scaling
Skew
Code regression
```

---

# 47. Retry Cost

Suppose:

```text
Normal run = $5
```

A persistent failure causes:

```text
Initial = $5
Retry 1 = $5
Retry 2 = $5
Retry 3 = $5
```

Potential compute cost:

```text
$20
```

before considering other factors.

Therefore excessive retries can become expensive.

---

# 48. Concurrency Cost

Increasing concurrency can improve:

```text
Runtime
```

but may also increase:

```text
Compute usage
Target load
Cost
```

Example:

```text
Concurrency 10
→ $5/run

Concurrency 50
→ $8/run

Concurrency 100
→ $15/run
```

If:

```text
10 → 3h
50 → 1.5h
100 → 1.4h
```

then 100 may not provide enough additional value to justify the cost.

---

# 49. Cost + SLA Decision

Use:

```text
SLA
+
Runtime
+
Cost
```

Example:

| Concurrency | Runtime | Cost | SLA |
|---:|---:|---:|---|
| 10 | 3h | $5 | ❌ |
| 25 | 2h 10m | $6 | ❌ |
| 50 | 1h 40m | $8 | ✅ |
| 100 | 1h 35m | $14 | ✅ |

Best candidate:

```text
50
```

because it meets the SLA at a lower cost than 100.

---

# 50. Cost + Reliability

The cheapest configuration is not necessarily the best.

Suppose:

```text
Cheap compute
→ frequent OOM
→ retries
→ SLA breaches
```

The effective cost can be higher.

Therefore evaluate:

```text
Nominal compute cost
+
Retry cost
+
Failure cost
+
Operational cost
+
SLA impact
```

---

# 51. Compute and Failure Recovery

Suppose a large Job repeatedly fails because of insufficient memory.

Increasing retries:

```text
Retry
Retry
Retry
```

does not solve the underlying problem.

Better:

```text
Identify OOM
   ↓
Increase memory / optimize workload
   ↓
Test
   ↓
Run
```

### Professional rule

> **Use retries for transient failures, not as a substitute for capacity correction.**

---

# 52. Compute and Data Skew

Suppose:

```text
100 partitions
```

but one partition contains:

```text
70% of the data
```

Adding more workers may have limited benefit.

Investigate:

```text
Partitioning
Join strategy
Data distribution
Skew handling
```

before simply scaling out.

---

# 53. Compute and Shuffle

Shuffle-heavy workloads can become bottlenecked by:

```text
Memory
Disk I/O
Network
Partition distribution
```

Increasing CPU alone may not solve the problem.

---

# 54. Compute and Small Files

Large numbers of small files can create:

```text
File listing overhead
Task scheduling overhead
Metadata overhead
Poor I/O efficiency
```

The correct optimization may be:

```text
Optimize file layout
```

rather than:

```text
Add more workers
```

---

# 55. Compute and Workload Type

Different workloads have different resource requirements.

### CPU-heavy

Need:

```text
CPU capacity
```

### Memory-heavy

Need:

```text
Memory capacity
```

### I/O-heavy

Need:

```text
Efficient storage/network I/O
```

### Shuffle-heavy

Need:

```text
Memory + network + appropriate partitioning
```

---

# 56. Compute Decision Framework

When selecting compute:

```text
1. What task type is this?

2. Is serverless supported?

3. Is serverless appropriate?

4. Is low startup latency important?

5. Is this a batch workload?

6. What are CPU/memory/I/O requirements?

7. Is the workload variable?

8. Should compute be shared?

9. What is the SLA?

10. What is the cost target?

11. Is accurate cost attribution required?
```

---

# 57. Concurrency Decision Framework

When selecting concurrency:

```text
1. Are workloads independent?

2. Can writes safely overlap?

3. What is the source capacity?

4. What is the target capacity?

5. What is the compute capacity?

6. What is the workspace limit?

7. What is the SLA?

8. What does benchmarking show?

9. What does the incremental concurrency cost?

10. Is the additional performance meaningful?
```

---

# 58. Cost Decision Framework

When optimizing cost:

```text
1. Measure current cost.

2. Identify largest cost drivers.

3. Identify slow tasks.

4. Check unnecessary retries.

5. Check oversized compute.

6. Check excessive concurrency.

7. Check serverless performance mode.

8. Check compute sharing opportunities.

9. Benchmark alternatives.

10. Choose the configuration that meets SLA at acceptable cost.
```

---

# 59. Professional Scenario: SLA

### Requirement

```text
Job must finish within 2 hours.
```

Testing:

```text
Concurrency 20 → 2h 45m
Concurrency 50 → 1h 50m
Concurrency 100 → 1h 45m
```

Correct reasoning:

```text
50 already meets SLA
```

If:

```text
100 costs significantly more
```

choose:

```text
50
```

assuming other constraints are satisfied.

---

# 60. Professional Scenario: Overlapping Runs

### Requirement

```text
Job runs every 15 minutes.
Runtime = 45 minutes.
```

Question:

> Should concurrent runs be allowed?

Answer depends on:

```text
Are runs independent?
Are writes idempotent?
Can source/target handle overlap?
Is ordering required?
```

If yes:

```text
Increase max concurrent runs
```

may be appropriate.

If no:

```text
Keep concurrency controlled
```

and consider queueing.

---

# 61. Professional Scenario: For Each

### Requirement

```text
500 independent folders
```

Testing:

```text
Concurrency 10 → 3h
Concurrency 50 → 1h 40m
Concurrency 100 → 1h 35m
```

SLA:

```text
2h
```

Best reasoning:

```text
50
```

if it meets SLA with acceptable cost/capacity.

---

# 62. Professional Scenario: Serverless

### Requirement

```text
Standard nightly batch
```

Startup latency:

```text
Not critical
```

Current serverless options:

```text
Performance-optimized
Standard
```

Consider:

```text
Standard
```

because it is designed for automated batch workloads where longer startup is acceptable and can reduce cost.

---

# 63. Professional Scenario: Unsupported Serverless Workload

### Requirement

```text
JAR task
```

Serverless does not support the required workload.

Correct direction:

```text
Classic Jobs compute
```

Do not force serverless.

---

# 64. Professional Scenario: Cost Attribution

### Requirement

> Precisely determine Job-level compute cost.

Prefer:

```text
Serverless
```

or:

```text
Dedicated Jobs compute
```

rather than shared all-purpose compute.

---

# 65. Professional Scenario: High Concurrency

Suppose:

```text
Job concurrency = 5
For Each concurrency = 100
```

Potential task-level parallelism:

```text
≈ 500 iteration tasks
```

Before accepting this configuration, evaluate:

```text
Workspace limit
Source capacity
Target capacity
Compute
Cost
```

The answer is not automatically:

```text
500 is good
```

---

# 66. Production Compute Architecture

A production workflow might look like:

```text
                    JOB
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      Task Group A          Task Group B
          ↓                     ↓
   Serverless / Job       Specialized Compute
       Compute                 Compute
          │                     │
          └──────────┬──────────┘
                     ↓
                  Outputs
```

Choose compute based on workload rather than forcing every task onto identical infrastructure.

---

# 67. Compute + Concurrency + Cost

These three concepts must be evaluated together.

```text
Compute
   ↓
How much capacity?

Concurrency
   ↓
How much parallel work?

Cost
   ↓
What does that capacity/parallelism cost?
```

Example:

```text
More workers
+
More concurrent iterations
=
Faster
```

but potentially:

```text
Higher cost
+
Higher source/target pressure
+
Higher failure probability
```

---

# 68. The Professional Optimization Loop

Use:

```text
MEASURE
   ↓
IDENTIFY BOTTLENECK
   ↓
CHANGE ONE THING
   ↓
BENCHMARK
   ↓
CHECK SLA
   ↓
CHECK COST
   ↓
CHECK RELIABILITY
   ↓
KEEP / REVERT
```

This is better than:

```text
"Just increase the cluster."
```

---

# 69. Exam Decision Rules

### Rule 1

> **Use serverless compute for supported workloads when it is appropriate; Databricks currently recommends serverless for most Job workloads.**

### Rule 2

> **Use Jobs compute rather than all-purpose compute for production Jobs.**

### Rule 3

> **Shared compute can reduce startup latency, but separate compute may be better for incompatible workloads.**

### Rule 4

> **Maximum concurrent runs controls concurrent runs of the same Job.**

### Rule 5

> **For Each concurrency controls parallel iterations, not Job Runs.**

### Rule 6

> **Workspace-level task concurrency is a separate constraint.**

### Rule 7

> **Higher concurrency is not automatically better.**

### Rule 8

> **Only allow overlapping Job Runs when workload correctness and infrastructure capacity support it.**

### Rule 9

> **Use queueing when runs should wait rather than be skipped, subject to queueing behavior and limits.**

### Rule 10

> **Choose the smallest tested concurrency that satisfies the SLA and acceptable cost.**

### Rule 11

> **Use billing/system tables to validate actual cost rather than relying only on theoretical estimates.**

### Rule 12

> **Fix capacity/performance problems rather than relying on retries to compensate for them.**

---

# 70. Common Exam Traps

## Trap 1

> More concurrency is always faster.

❌ Incorrect.

The source/target/compute may become the bottleneck.

---

## Trap 2

> Job concurrency = For Each concurrency.

❌ Incorrect.

```text
Job concurrency
→ Job Runs

For Each concurrency
→ Iterations
```

---

## Trap 3

> Serverless supports every task.

❌ Incorrect.

Some workloads require classic compute.

---

## Trap 4

> Cheapest compute is always best.

❌ Incorrect.

The workload must satisfy:

```text
Correctness
SLA
Reliability
Cost
```

---

## Trap 5

> All-purpose compute is preferred for production Jobs.

❌ Incorrect.

Databricks recommends Jobs compute/serverless for production Job workloads.

---

## Trap 6

> If a Job is slow, increase workers immediately.

❌ Incorrect.

First identify the bottleneck.

---

## Trap 7

> More Job concurrency automatically improves throughput.

❌ Incorrect.

Overlapping runs may create:

```text
Write conflicts
Resource contention
Duplicate processing
Higher cost
```

---

## Trap 8

> Standard serverless mode is always better because it is cheaper.

❌ Incorrect.

If startup latency is business-critical, Performance-optimized may be appropriate.

---

# 71. Quick Reference

| Requirement | Consider |
|---|---|
| Supported normal Job workload | Serverless |
| JAR/Spark Submit | Classic Jobs compute |
| Production classic Job | Jobs compute |
| Interactive workload | All-purpose may be appropriate |
| Shared compatible tasks | Shared Job compute |
| Different workload profiles | Separate compute |
| Fast serverless startup | Performance-optimized |
| Cost-focused batch | Standard serverless |
| Multiple Job Runs | Max concurrent runs |
| Parallel For Each inputs | For Each concurrency |
| Prevent skipped runs | Queueing |
| Accurate Job cost | Serverless / dedicated Job compute |
| Cost analysis | `system.billing.usage` + Lakeflow tables |
| Slow Job | Measure bottleneck first |
| SLA optimization | Benchmark concurrency/compute |
| Variable workload | Consider autoscaling |

---

# 72. Current Important Limits

Current Databricks documentation states:

```text
Default max concurrent Job Runs = 1

Maximum max concurrent Job Runs = 1000

Workspace concurrent task runs = 2000

Maximum saved Jobs per workspace = 12,000

Maximum tasks per Job = 1000
```

These are current documented platform limits and can change.

Always verify current documentation before the exam.

Official documentation:

https://docs.databricks.com/aws/en/jobs/

---

# 73. Final Mental Model

```text
                     JOB
                      │
               ┌──────┴──────┐
               ↓             ↓
            COMPUTE      CONCURRENCY
               │             │
        ┌──────┴──────┐      ├── Job Runs
        ↓             ↓      ├── For Each
    Serverless      Classic   └── Workspace
        │             │
        └──────┬──────┘
               ↓
           PERFORMANCE
               │
        ┌──────┴──────┐
        ↓             ↓
       SLA           COST
        │             │
        └──────┬──────┘
               ↓
           OPTIMIZE
```

---

# 74. One-Line Rules to Memorize

```text
Serverless
→ Default/recommended choice for most supported Job workloads.

Jobs compute
→ Preferred classic compute for production Jobs.

All-purpose compute
→ Not recommended for production Jobs.

Shared compute
→ Can reduce startup overhead.

Job concurrency
→ Concurrent Job Runs.

For Each concurrency
→ Concurrent iterations.

Workspace concurrency
→ Overall task-run capacity.

Queueing
→ Wait instead of skipping when applicable.

More concurrency
→ Not automatically better.

Performance mode
→ Balance startup latency and cost.

SLA
→ Minimum required performance target.

Cost optimization
→ Meet SLA at acceptable cost.

Bottleneck
→ Optimize the limiting resource, not blindly everything.

Benchmark
→ Measure before choosing production configuration.
```

---

# 75. Official Documentation

Use these as the primary references for this chapter:

- [Configure compute for jobs](https://docs.databricks.com/aws/en/jobs/compute)
- [Configure and edit Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/configure-job)
- [Configure classic compute for Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/run-classic-jobs)
- [Best practices for serverless compute](https://docs.databricks.com/aws/en/compute/serverless/best-practices)
- [Connect to serverless compute](https://docs.databricks.com/aws/en/compute/serverless)
- [Monitor Job costs and performance with system tables](https://docs.databricks.com/aws/en/admin/system-tables/jobs-cost)
- [Monitor serverless compute cost](https://docs.databricks.com/aws/en/admin/system-tables/serverless-billing)
- [Lakeflow Jobs overview and limits](https://docs.databricks.com/aws/en/jobs/)

---

## Chapter Status

**09. Compute, Concurrency & Cost — COMPLETE ✅**

### Key verified facts

```text
Databricks recommendation
→ Serverless for most supported Job workloads

Classic production Jobs
→ Prefer Jobs compute over all-purpose compute

Default max concurrent Job Runs
→ 1

Maximum max concurrent Job Runs
→ 1000

Workspace concurrent task runs
→ 2000

Queueing
→ Can queue runs instead of skipping when capacity/concurrency limits are reached

Queue duration
→ Up to 48 hours under documented conditions

For Each concurrency
→ Separate from Job Run concurrency

Serverless performance modes
→ Performance-optimized
→ Standard

Standard
→ Designed for automated batch workloads where startup latency can be tolerated

Cost monitoring
→ system.billing.usage
→ Lakeflow system tables

Accurate Job cost attribution
→ Easier with serverless or dedicated Job compute
```

