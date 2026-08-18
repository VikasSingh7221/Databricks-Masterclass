# 01. Lakeflow Jobs Fundamentals

> **Databricks Data Engineer Professional — Certification Notes**

---

## 1. What is Lakeflow Jobs?

**Lakeflow Jobs** is Databricks' workflow orchestration capability used to automate and coordinate data-processing tasks.

A Job defines:

- What work needs to be performed
- The order/dependencies between tasks
- When the workflow should start
- What parameters should be supplied
- How failures should be handled
- How tasks can execute conditionally or in parallel
- How the workflow can be monitored and recovered

A Job can contain multiple tasks that together form a workflow.

### Basic model

```text
Job
│
├── Task A
├── Task B
├── Task C
└── Task D
```

The tasks and their dependencies form a workflow/DAG.

---

# 2. Job vs Task vs Job Run

These three concepts must be clearly separated.

## Job

A **Job** is the workflow definition/configuration.

Example:

```text
Customer_Daily_Pipeline
```

It defines:

- Tasks
- Dependencies
- Parameters
- Triggers
- Compute
- Notifications
- Retry behavior
- Concurrency settings

Think:

> **Job = the workflow definition**

---

## Task

A **Task** is an individual unit of work inside a Job.

Examples:

```text
Extract Customers
Validate Data
Transform Data
Load Gold
Send Alert
```

Think:

> **Task = one unit of work**

---

## Job Run

A **Job Run** is one execution of a Job.

Example:

```text
Job:
Customer_Daily_Pipeline

Runs:
Run #1001
Run #1002
Run #1003
```

The same Job can execute many times.

Think:

> **Job = definition**

> **Job Run = one execution of that definition**

---

# 3. Lakeflow Jobs Mental Model

The complete workflow can be understood as:

```text
                    JOB
                     │
              ┌──────┴──────┐
              │             │
           Trigger      Parameters
              │             │
              └──────┬──────┘
                     ↓
                  Job Run
                     ↓
                   Tasks
                     ↓
               Dependencies
                     ↓
                Run If / 
                 Branching
                     ↓
                 For Each
                     ↓
                Concurrency
                     ↓
              Retry / Recovery
                     ↓
               Notifications
```

---

# 4. DAG — Directed Acyclic Graph

Tasks in a Job can be connected using dependencies.

Example:

```text
A → B → C
```

This means:

```text
A must complete before B
B must complete before C
```

Another example:

```text
       ┌──→ B ──┐
A ─────┤        ├──→ D
       └──→ C ──┘
```

Here:

- A must complete before B and C
- B and C can execute independently
- D depends on B and C

### Professional principle

> **Dependencies should represent real data/business dependencies.**

Do not create unnecessary sequential dependencies just to control execution order.

---

# 5. Parallelism

If tasks are independent:

```text
A ──┐
    ├──→ D
B ──┘
```

A and B can potentially execute in parallel.

If you unnecessarily define:

```text
A → B → D
```

then B must wait for A even if there is no real dependency.

This can increase:

- Runtime
- SLA risk
- Compute duration

### Professional decision

Before adding a dependency, ask:

> **Does Task B actually require the output/state of Task A?**

If not, they may be independent.

---

# 6. Job Lifecycle

A simplified lifecycle:

```text
Job configured
      ↓
Trigger / manual execution
      ↓
Job Run created
      ↓
Tasks evaluated
      ↓
Dependencies evaluated
      ↓
Tasks execute
      ↓
Success / Failure / Exclusion
      ↓
Downstream tasks evaluated
      ↓
Job Run completes
```

The exact behavior depends on task dependencies, `Run If` conditions, retries, cancellations, and other configuration.

---

# 7. Trigger vs Job

A common Professional-level distinction:

### Job

Defines:

> **What should happen?**

### Trigger

Defines:

> **When should the Job start?**

Example:

```text
Job:
Customer Pipeline

Trigger:
Every day at 2 AM
```

The trigger starts a Job Run according to its configured condition/schedule.

---

# 8. Trigger vs Dependency

These must NOT be confused.

```text
Trigger
   ↓
When should the Job start?
```

versus:

```text
Dependency
   ↓
When can the downstream task execute?
```

Example:

```text
2 AM
 ↓
Job starts
 ↓
Task A
 ↓
Task B
```

The 2 AM schedule starts the Job.

The dependency:

```text
A → B
```

controls when B can execute.

---

# 9. Task Execution Model

Suppose:

```text
A ──→ B ──→ C
```

The general execution flow is:

```text
A starts
 ↓
A completes
 ↓
B becomes eligible
 ↓
B executes
 ↓
B completes
 ↓
C becomes eligible
 ↓
C executes
```

For a more complex DAG:

```text
A ──┐
    ├──→ C
B ──┘
```

C requires the configured upstream conditions to be satisfied before it executes.

---

# 10. Independent Tasks

Suppose:

```text
Salesforce Ingestion ──┐
                       ├──→ Validation
Snowflake Ingestion ───┘
```

Salesforce and Snowflake are independent.

Therefore:

```text
Salesforce
     │
     └─────────┐
               ↓
           Validation
               ↑
     ┌─────────┘
     │
Snowflake
```

They can potentially run in parallel.

Validation waits for its required upstream conditions.

---

# 11. Job-Level vs Task-Level Configuration

Some settings apply to the Job as a whole.

Examples:

```text
Job Parameters
Triggers
Maximum concurrent runs
Notifications
Queueing
```

Other configuration belongs to individual tasks.

Examples:

```text
Task parameters
Task dependencies
Task retries
Task-specific compute
Run If
```

Always identify the **scope** of a configuration setting.

---

# 12. Job Parameters

Job Parameters provide configuration to the Job.

Examples:

```text
environment = prod
processing_date = 2026-08-17
batch_size = 5000
```

Use them for values that represent Job/run configuration.

### Good examples

```text
environment
processing_date
region
batch_size
source_system
```

Mental model:

```text
Configuration
      ↓
Job Parameter
      ↓
Tasks
```

---

# 13. Task Values

Task Values are different.

A task can generate a value during execution.

Example:

```python
row_count = 125000

dbutils.jobs.taskValues.set(
    key="row_count",
    value=row_count
)
```

Another task can consume that runtime-generated value.

Mental model:

```text
Task A
   ↓
calculates value
   ↓
Task Value
   ↓
Dynamic reference
   ↓
Task B
```

### Key distinction

```text
Job Parameter
→ configuration/input

Task Value
→ runtime-generated output
```

---

# 14. For Each

For Each is useful when the same processing logic must be applied to many inputs.

Example:

```text
500 folders
    ↓
For Each
    ↓
Same processing logic
```

Instead of creating:

```text
Task 1
Task 2
Task 3
...
Task 500
```

use one reusable task inside a For Each construct.

---

# 15. Concurrency

Concurrency controls how much work can execute in parallel.

There are multiple levels.

## Job Run concurrency

Controls:

> How many Job Runs can overlap?

```text
Run #1 ─────────────
Run #2 ─────────────
```

---

## For Each concurrency

Controls:

> How many For Each iterations can execute concurrently?

Example:

```text
1000 folders
For Each concurrency = 100
```

means controlled parallel execution of iterations.

### Important distinction

```text
Job concurrency
→ parallelism BETWEEN Job Runs

For Each concurrency
→ parallelism WITHIN a Job Run
```

---

# 16. Retry

Retries are useful for failures that may be transient.

Example:

```text
Network timeout
      ↓
Retry
      ↓
Success
```

Retries should not be used blindly.

For deterministic failures such as:

```text
Permission denied
Invalid SQL
Invalid configuration
```

the underlying issue should generally be fixed first.

---

# 17. Idempotency

A production workflow should consider what happens if a task executes more than once.

Example:

```text
Attempt 1
   ↓
Partial write
   ↓
Failure

Retry
   ↓
Same input processed again
```

If the write is not retry-safe, duplicates may occur.

### Unsafe example

```sql
INSERT INTO target
SELECT ...
```

Repeated execution can create duplicate records depending on the target design.

### Safer design

Use a deterministic strategy based on stable business identity, such as an appropriate:

```text
MERGE
Overwrite
Delete + Insert
Deduplication
```

The exact strategy depends on the data model.

### Important

> **MERGE does not automatically guarantee idempotency.**

The matching condition must be deterministic and based on a stable logical identity.

---

# 18. Repair and Recovery

When only part of a workflow fails, avoid unnecessarily reprocessing successful work.

Example:

```text
Folder 1–126   → SUCCESS
Folder 127     → FAILED
Folder 128–500 → SUCCESS
```

Preferred recovery:

```text
Fix root cause
      ↓
Verify retry safety
      ↓
Repair Folder 127
      ↓
Validate
```

Avoid:

```text
Rerun all 500 folders
```

unless the architecture/business requirement requires a full rerun.

### Professional principle

> **Recover the smallest safe unit of failed work.**

---

# 19. Monitoring

Monitoring should consider more than Job status.

Three important dimensions:

```text
Correctness
Performance
Reliability
```

### Correctness

```text
Row counts
Data quality
Schema validation
Business rules
```

### Performance

```text
Runtime
CPU
Memory
I/O
Shuffle
Throughput
```

### Reliability

```text
Failures
Retries
Skipped/excluded tasks
SLA compliance
Failure trends
```

---

# 20. Notifications

Notifications should provide useful operational information.

Examples:

```text
Job failed
Critical task failed
Validation failed
Retries exhausted
SLA/duration threshold exceeded
```

Avoid excessive notifications.

Bad:

```text
100 tasks
100 success emails
100 retry emails
```

Better:

```text
Critical failure → alert
Retries exhausted → alert
Business validation failure → alert
SLA degradation → warning
```

### Professional principle

> **Alert on actionable events, not every event.**

---

# 21. Compute

Tasks require compute resources.

The correct compute choice depends on:

- Workload type
- SQL vs Spark/Python workload
- Runtime requirements
- Performance
- Cost
- Scaling requirements
- Supported task type

### SQL-heavy workload

A SQL Warehouse may be appropriate.

### Spark/Python-heavy workload

Job compute/serverless compute may be more appropriate depending on the workload and supported configuration.

### Important

Do not choose compute merely because it is "bigger."

Use:

```text
Measure
   ↓
Identify bottleneck
   ↓
Optimize
   ↓
Measure again
   ↓
Scale if justified
```

---

# 22. Performance Optimization

When a Job is slow:

### Do NOT immediately:

```text
Increase compute
Increase concurrency
Increase retries
```

First investigate the bottleneck.

Potential causes:

```text
Data skew
Poor join strategy
Too much data scanned
Unnecessary columns
Poor partitioning
Shuffle
Resource contention
Source/target bottleneck
```

Then:

```text
Optimize
   ↓
Measure
   ↓
Scale if necessary
```

---

# 23. Incremental Processing

If a pipeline processes:

```text
10 TB historical data
20 GB new data
```

every night, investigate incremental processing.

Instead of:

```text
Full refresh
10 TB
 ↓
Every night
```

prefer where appropriate:

```text
New/changed data
20 GB
 ↓
Incremental processing
```

This can reduce:

- Runtime
- Compute
- Cost
- Data scanned

---

# 24. Late-Arriving Data

Suppose:

```text
99% of late data → within 3 days
1% → up to 30 days
```

A common architecture is:

```text
Normal processing
      ↓
3-day lookback

Exceptional historical corrections
      ↓
Targeted backfill
```

Avoid processing the entire 30-day window every night if the business requirement doesn't require it.

---

# 25. SLA Thinking

Suppose:

```text
Normal runtime = 1.5 hours
SLA = 2 hours
Today = 1.8 hours
```

The Job is still within SLA.

Do not automatically classify it as a production failure.

Instead investigate if there is a meaningful degradation trend.

Example:

```text
1.5h
 ↓
1.7h
 ↓
1.8h
 ↓
1.9h
 ↓
2.1h
```

This indicates an emerging performance problem.

---

# 26. Production Architecture

A mature Job may look like:

```text
                Trigger
                   ↓
             Job Parameters
                   ↓
          ┌────────┴────────┐
          ↓                 ↓
      Source A           Source B
          ↓                 ↓
          └────────┬────────┘
                   ↓
               Validation
                   ↓
                Run If
                   ↓
                For Each
                   ↓
          Controlled Concurrency
                   ↓
            Processing Tasks
                   ↓
         ┌─────────┴─────────┐
         ↓                   ↓
      Success              Failure
                             ↓
                          Retry
                             ↓
                     Retry exhausted
                             ↓
                          Alert
                             ↓
                         Repair
```

---

# 27. Professional Mental Model

When solving a scenario, ask these questions in order:

```text
1. WHEN?
   → Trigger

2. WHAT CONFIGURATION?
   → Job Parameters

3. WHAT DEPENDS ON WHAT?
   → Dependencies

4. SHOULD IT EXECUTE?
   → Run If / Control Flow

5. MANY INPUTS?
   → For Each

6. HOW MUCH PARALLELISM?
   → Concurrency

7. WHAT IF IT FAILS?
   → Retry / Failure handling

8. CAN IT BE RETRIED SAFELY?
   → Idempotency

9. WHAT IF IT STILL FAILS?
   → Repair / Recovery

10. WHO NEEDS TO KNOW?
    → Notifications

11. IS IT FAST ENOUGH?
    → SLA / Performance

12. IS IT COST-EFFICIENT?
    → Compute / Optimization
```

---

# 28. Certification Focus

For Professional-level questions, don't memorize isolated features.

Focus on **trade-offs**:

```text
Performance vs Cost
Parallelism vs Resource contention
Retry vs Retry safety
Full refresh vs Incremental processing
Automation vs Alert fatigue
Independence vs Dependency
Availability vs Data correctness
Fast recovery vs Duplicate risk
```

The best answer is usually the one that:

1. Preserves correctness
2. Minimizes unnecessary processing
3. Handles failures safely
4. Meets the SLA
5. Controls cost
6. Is operationally maintainable

---

# 29. Quick Reference

| Concept | Remember |
|---|---|
| Job | Workflow definition |
| Job Run | One execution |
| Task | Unit of work |
| Dependency | Execution relationship |
| Trigger | Starts Job Run |
| Run If | Conditional execution |
| Job Parameter | Job/run configuration |
| Task Value | Runtime-generated value |
| Dynamic Reference | References runtime metadata/value |
| For Each | Repeated processing |
| For Each concurrency | Parallel iterations |
| Job concurrency | Parallel Job Runs |
| Retry | Transient failure handling |
| Backoff | Avoid retry storms |
| Idempotency | Safe repeated execution |
| Repair | Targeted recovery |
| Validation | Correctness/business rules |
| Notification | Operational visibility |
| SLA | Required completion target |
| Incremental | Process only relevant new/changed data |
| Backfill | Historical correction/reprocessing |

---

# 30. Official Documentation

For certification preparation, verify product-specific behavior against current Databricks documentation:

- [Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/)
- [Configure jobs](https://docs.databricks.com/aws/en/jobs/configure-job)
- [Run If conditions](https://docs.databricks.com/aws/en/jobs/run-if)
- [Job parameters](https://docs.databricks.com/aws/en/jobs/job-parameters)
- [Parameters](https://docs.databricks.com/aws/en/jobs/parameters)
- [Dynamic value references](https://docs.databricks.com/aws/en/jobs/dynamic-value-references)
- [For Each](https://docs.databricks.com/aws/en/jobs/tasks/for-each)
- [Triggers](https://docs.databricks.com/aws/en/jobs/triggers)
- [File-arrival triggers](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers)
- [Task configuration](https://docs.databricks.com/aws/en/jobs/configure-task)
- [Notifications](https://docs.databricks.com/aws/en/jobs/notifications)
- [Compute](https://docs.databricks.com/aws/en/jobs/compute)

---

