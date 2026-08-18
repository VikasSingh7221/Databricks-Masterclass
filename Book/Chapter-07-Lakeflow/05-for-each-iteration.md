# 05. For Each & Iteration

> **Databricks Data Engineer Professional — Certification Notes**

---

# 1. What is For Each?

The **For Each task** is used when the same processing logic needs to be executed for multiple input values.

Instead of creating many individual tasks:

```text
Task 1 → Folder 1
Task 2 → Folder 2
Task 3 → Folder 3
...
Task 500 → Folder 500
```

use:

```text
For Each
   ↓
Input list
   ↓
Same nested task
   ↓
Iteration 1
Iteration 2
Iteration 3
...
Iteration 500
```

### Mental model

> **For Each = apply the same task logic to multiple inputs.**

Official documentation:

https://docs.databricks.com/aws/en/jobs/tasks/for-each

---

# 2. Why Use For Each?

For Each is useful when:

- The same logic must run for many files
- The same logic must run for many folders
- The same logic must run for multiple tables
- The same logic must run for multiple partitions
- The same processing must run for a list of IDs
- Work items are discovered dynamically at runtime

Example:

```text
500 folders
      ↓
Same processing logic
      ↓
For Each
```

---

# 3. Basic Structure

Conceptually:

```text
               For Each
                  │
             Input list
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
   Iteration 1  Iteration 2  Iteration 3
       │          │          │
       └──────────┴──────────┘
                  ↓
             Same nested task
```

The nested task contains the processing logic.

---

# 4. Example — Folder Processing

Suppose:

```text
folders = [
    "customer_001",
    "customer_002",
    "customer_003",
    "customer_004"
]
```

For Each processes:

```text
customer_001
customer_002
customer_003
customer_004
```

using the same nested task.

Conceptually:

```text
For Each
   ↓
folder_001 → Process Folder
folder_002 → Process Folder
folder_003 → Process Folder
folder_004 → Process Folder
```

---

# 5. Why Not Create Separate Tasks?

Without For Each:

```text
Task_001
Task_002
Task_003
...
Task_500
```

This creates unnecessary workflow complexity.

With For Each:

```text
For Each
   ↓
Process Folder
```

Advantages:

- Less DAG complexity
- Reusable processing logic
- Easier maintenance
- Easier scaling
- Controlled concurrency
- Easier recovery design

### Professional principle

> **Use For Each when the processing logic is the same and only the input changes.**

---

# 6. Input to For Each

For Each operates over a collection/list of inputs.

Example:

```text
[
  "folder_001",
  "folder_002",
  "folder_003"
]
```

Each item becomes an iteration input.

Conceptually:

```text
Input list
    ↓
For Each
    ↓
item 1
item 2
item 3
```

---

# 7. Iteration Input

Inside the nested task, the current iteration's input can be referenced using the For Each input reference.

Conceptually:

```text
{{input}}
```

For example:

```text
Input:
customer_001

Nested task:
process {{input}}
```

The next iteration receives:

```text
customer_002
```

and the same task processes it.

---

# 8. Object Inputs

For Each inputs don't have to be simple strings.

You may have structured objects such as:

```json
[
  {
    "folder": "customer_001",
    "region": "US"
  },
  {
    "folder": "customer_002",
    "region": "EU"
  }
]
```

Then the nested task can use properties of the current input.

Conceptually:

```text
{{input.folder}}
{{input.region}}
```

### Professional principle

> **Use structured inputs when each iteration needs multiple related values.**

---

# 9. Static vs Dynamic Input

For Each input can be known beforehand:

```text
[
  "A",
  "B",
  "C"
]
```

or generated dynamically by an upstream task.

### Static

```text
Job configuration
      ↓
For Each
```

### Dynamic

```text
Discover Work
      ↓
Task Value
      ↓
Dynamic Reference
      ↓
For Each
```

The dynamic pattern is particularly important for production workflows.

---

# 10. Dynamic For Each Pattern

Suppose a task discovers folders.

```text
Discover Folders
```

It generates:

```text
[
  "folder_001",
  "folder_002",
  "folder_003"
]
```

The task stores the list as a Task Value.

```python
dbutils.jobs.taskValues.set(
    key="folders",
    value=folders
)
```

The For Each task can reference the Task Value dynamically.

Conceptually:

```text
{{tasks.discover_folders.values.folders}}
```

Architecture:

```text
Discover Folders
       ↓
Task Value: folders
       ↓
Dynamic Reference
       ↓
For Each
       ↓
Iterations
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/tasks/for-each

---

# 11. For Each Concurrency

Concurrency controls how many iterations can execute at the same time.

Example:

```text
1000 folders
Concurrency = 100
```

Conceptually:

```text
100 iterations
      ↓
execute concurrently
      ↓
next available iterations
      ↓
continue until all 1000 complete
```

### Important

For Each concurrency is **not** Job Run concurrency.

---

# 12. For Each Concurrency vs Job Run Concurrency

This is a major Professional exam distinction.

## For Each concurrency

Controls:

> **How many iterations run concurrently inside a Job Run?**

Example:

```text
Job Run
   ↓
For Each
   ↓
Concurrency = 50
```

---

## Job Run concurrency

Controls:

> **How many Job Runs can execute concurrently?**

Example:

```text
Run #1 ─────────────
Run #2 ─────────────
```

### Remember

```text
For Each concurrency
→ parallelism INSIDE one Job Run

Job concurrency
→ parallelism BETWEEN Job Runs
```

---

# 13. Default For Each Concurrency

Databricks currently documents the default For Each concurrency as:

```text
1
```

This means iterations execute sequentially unless concurrency is configured otherwise.

Official documentation:

https://docs.databricks.com/aws/en/jobs/tasks/for-each

### Example

```text
100 folders
Concurrency = 1
```

means one iteration at a time.

---

# 14. Choosing Concurrency

Do not assume:

```text
Higher concurrency = always better
```

Instead evaluate:

```text
Source capacity
Target capacity
Compute capacity
Memory
CPU
Network
Shuffle
SLA
Cost
```

Example:

```text
Concurrency = 10
Runtime = 3 hours

Concurrency = 50
Runtime = 1.5 hours

Concurrency = 200
Runtime = 1.4 hours
```

If 50 and 200 have almost the same runtime:

```text
50
```

may be the better operational choice.

### Professional principle

> **Choose concurrency based on measured benefit and resource capacity, not maximum possible parallelism.**

---

# 15. Concurrency and Resource Bottlenecks

Suppose:

```text
Concurrency = 100
```

but the target database can handle only 30 concurrent writes efficiently.

Increasing to 200 may make performance worse.

```text
More parallelism
       ↓
More target contention
       ↓
Slower writes
       ↓
Longer runtime
```

Therefore:

> **Parallelism must match the bottleneck.**

---

# 16. Concurrency and SLA

Suppose:

```text
Current runtime = 2h 30m
SLA = 2h
```

Testing:

```text
Concurrency = 20
Runtime = 2h 30m

Concurrency = 50
Runtime = 1h 40m

Concurrency = 100
Runtime = 1h 35m
```

If 50 provides enough SLA headroom and has lower cost/resource pressure:

```text
Concurrency = 50
```

may be preferable to 100.

---

# 17. Independent Iterations

For Each is especially useful when iterations are independent.

Example:

```text
Folder 1 → independent
Folder 2 → independent
Folder 3 → independent
```

If Folder 127 fails:

```text
Folder 127 → FAILED
```

the other folders can potentially continue.

Conceptually:

```text
Folder 126 → SUCCESS
Folder 127 → FAILED
Folder 128 → SUCCESS
```

### Important

> **Iteration independence does not mean the overall For Each operation is necessarily successful.**

One failed iteration can still affect the overall workflow outcome and downstream tasks.

---

# 18. One Iteration Fails

Suppose:

```text
500 folders
```

and:

```text
Folder 127 → FAILED
```

while:

```text
499 folders → SUCCESS
```

Do not automatically assume:

```text
All 500 failed
```

Instead:

```text
Individual iteration status
        ↓
Folder 127 = FAILED

Other iterations
        ↓
Can continue if independent
```

This is a key failure-isolation principle.

---

# 19. Retry Failed Iterations

Suppose Folder 127 fails because of:

```text
Temporary network timeout
```

A retry may be appropriate.

Conceptually:

```text
Folder 127
   ↓
Attempt 1 → FAILED
   ↓
Retry
   ↓
Attempt 2 → SUCCESS
```

You should not automatically rerun all successful folders.

---

# 20. Retry Safety

Before retrying an iteration, ask:

```text
Did the failed attempt partially write data?
```

If yes, repeated processing may cause:

```text
Duplicate records
```

unless the write is retry-safe.

Therefore:

```text
Failure
   ↓
Retry
   ↓
Idempotency?
```

must be considered.

---

# 21. Idempotent For Each Processing

Suppose:

```text
Folder 127
```

contains:

```text
10,000 records
```

First attempt:

```text
5,000 records written
5,000 records remaining
```

Then the task fails.

If the retry blindly inserts all 10,000:

```text
5,000 duplicate records
```

could occur.

Better approaches may include:

```text
MERGE
Delete + Insert
Deterministic overwrite
Checkpointing
Deduplication
```

depending on the target design.

---

# 22. MERGE and Idempotency

A common assumption:

> "We use MERGE, therefore retries are safe."

This is incomplete.

The MERGE condition must use a stable logical identity.

Good conceptual example:

```text
MERGE ON customer_id
```

if `customer_id` is stable and uniquely identifies the record.

Potentially unsafe:

```text
MERGE ON customer_id + current_timestamp
```

because the timestamp may change between executions.

### Rule

> **Idempotency comes from deterministic processing and stable identity, not merely from using MERGE.**

---

# 23. Failed Iteration Recovery

Suppose:

```text
Folder 1–126   → SUCCESS
Folder 127     → FAILED
Folder 128–500 → SUCCESS
```

Preferred strategy:

```text
Fix root cause
      ↓
Verify retry safety
      ↓
Recover Folder 127
```

Avoid unnecessarily reprocessing:

```text
Folders 1–126
Folders 128–500
```

---

# 24. Transient vs Permanent Failure

### Transient

Examples:

```text
Network timeout
Temporary service unavailable
Temporary infrastructure issue
```

Retry may make sense.

### Permanent

Examples:

```text
Permission denied
Invalid SQL
Invalid configuration
Missing required object
```

Repeated retries may not help.

### Professional decision

```text
Failure
   ↓
Classify
   ↓
Transient?
 ┌──┴──┐
YES    NO
 ↓      ↓
Retry  Fix root cause
        ↓
      Recover
```

---

# 25. Retry Storm

Suppose 100 iterations fail because an external system is temporarily unavailable.

If all 100 immediately retry:

```text
100 failures
     ↓
100 immediate retries
     ↓
External system still unhealthy
     ↓
More failures
```

This can make the outage worse.

Use controlled retry/backoff behavior where appropriate.

### Principle

> **Retries should reduce transient failure impact, not amplify infrastructure pressure.**

---

# 26. For Each + Validation

Suppose:

```text
Source A
Source B
   ↓
Validation
   ↓
For Each
```

If validation fails:

```text
For Each should not blindly process invalid data
```

The dependency and `Run If` condition should reflect the business requirement.

Example:

```text
Validation
Run If = All Succeeded

For Each
depends on Validation
```

---

# 27. For Each + Dynamic Inputs

A production pattern:

```text
Discover Work
      ↓
Validate Work List
      ↓
Task Value
      ↓
For Each
      ↓
Controlled concurrency
      ↓
Process each item
```

This is useful when the number of work items changes every day.

Example:

```text
Monday → 500 folders
Tuesday → 700 folders
Wednesday → 420 folders
```

No Job redesign is required.

---

# 28. For Each + Objects

Instead of passing:

```text
[
  "folder_1",
  "folder_2"
]
```

you can conceptually use structured objects:

```json
[
  {
    "folder": "folder_1",
    "region": "US",
    "priority": "high"
  },
  {
    "folder": "folder_2",
    "region": "EU",
    "priority": "normal"
  }
]
```

The nested task receives the current object.

This avoids having to maintain multiple unrelated lists.

---

# 29. For Each + Multiple Values

Suppose every iteration needs:

```text
folder
region
table
```

Instead of:

```text
folder_list
region_list
table_list
```

use structured iteration objects:

```json
[
  {
    "folder": "folder_1",
    "region": "US",
    "table": "customer_us"
  },
  {
    "folder": "folder_2",
    "region": "EU",
    "table": "customer_eu"
  }
]
```

### Benefit

The values remain associated with the same iteration.

---

# 30. For Each and Task Values

A common architecture:

```text
Task A
  ↓
generates list
  ↓
Task Value
  ↓
Dynamic Reference
  ↓
For Each
```

This allows runtime discovery.

Example:

```python
folders = discover_folders()

dbutils.jobs.taskValues.set(
    key="folders",
    value=folders
)
```

Then:

```text
For Each input:
{{tasks.discover_folders.values.folders}}
```

---

# 31. For Each and Parameters

Job parameters can provide global configuration:

```text
environment = prod
processing_date = 2026-08-17
```

For Each provides iteration-specific input:

```text
folder = customer_127
```

Therefore:

```text
Job Parameter
→ global/run-level configuration

For Each input
→ current iteration's work item
```

This separation is useful.

---

# 32. For Each and Job Parameters

Example:

```text
environment = prod
```

Every iteration can use the same environment configuration.

```text
For Each
 ├── folder_001 → prod
 ├── folder_002 → prod
 ├── folder_003 → prod
 └── folder_004 → prod
```

The folder changes.

The Job configuration remains the same.

---

# 33. For Each and Task Parameters

Task-specific configuration can also be combined with For Each.

Conceptually:

```text
For Each
   ↓
Current input = folder_001
   ↓
Nested task
   ↓
Task-specific configuration
```

The task can combine:

```text
Job Parameters
+
Task Parameters
+
For Each Input
```

depending on the task configuration.

---

# 34. For Each and Failure Isolation

This is a high-value Professional pattern.

Suppose:

```text
500 independent folders
```

and:

```text
Folder 127 → FAILED
```

The architecture should not unnecessarily stop:

```text
Folder 128–500
```

if they are independent and can safely continue.

This improves:

- Throughput
- Resource utilization
- Recovery time

---

# 35. For Each and Business Criticality

Not every failed iteration necessarily has the same business impact.

Example:

```text
Folder 1–499 → normal customers
Folder 500   → critical financial data
```

If Folder 500 fails, the downstream business rule may require stricter handling.

Therefore:

> **Iteration independence does not eliminate business-critical dependencies.**

---

# 36. For Each Anti-Patterns

## Anti-pattern 1 — Maximum concurrency

```text
1000 inputs
Concurrency = 1000
```

without testing.

### Problem

Can cause:

- Resource contention
- Target overload
- Network saturation
- Higher cost
- Worse performance

### Better

Benchmark and choose an appropriate concurrency.

---

# 37. Anti-pattern 2 — Sequential processing unnecessarily

```text
1000 inputs
Concurrency = 1
```

when the work is independent and the SLA requires parallelism.

### Problem

Long runtime.

### Better

Use controlled parallelism.

---

# 38. Anti-pattern 3 — Rerunning everything

```text
499 success
1 failure
   ↓
Rerun all 500
```

### Problem

Unnecessary work and potential duplicate processing.

### Better

Recover the failed iteration when safe.

---

# 39. Anti-pattern 4 — Blind retries

```text
Permission denied
   ↓
Retry
   ↓
Retry
   ↓
Retry
```

### Problem

The underlying issue remains.

### Better

Fix the root cause, then recover.

---

# 40. Anti-pattern 5 — Non-idempotent processing

```text
INSERT
 ↓
failure
 ↓
retry
 ↓
duplicate
```

### Better

Design the target write for safe reprocessing.

---

# 41. Anti-pattern 6 — Passing huge data through Task Values

Don't use:

```text
Task Value
```

as a replacement for a data storage system.

For large datasets:

```text
Delta / storage
```

and pass:

```text
path / table / identifier
```

instead.

---

# 42. For Each Performance Framework

When For Each is slow:

```text
1. Check input count
2. Check concurrency
3. Check source bottleneck
4. Check target bottleneck
5. Check compute utilization
6. Check data skew
7. Check network
8. Check task runtime distribution
9. Test concurrency changes
10. Measure again
```

Don't automatically increase concurrency.

---

# 43. Data Skew

Suppose:

```text
1000 folders
```

but one folder contains:

```text
70% of the total data
```

Then:

```text
999 folders → finish quickly
1 folder → remains running
```

Increasing overall concurrency may not solve the main problem.

The large/hot work item may need separate optimization.

### Professional principle

> **Parallelism cannot fully solve an uneven workload distribution.**

---

# 44. For Each and SLA

Suppose:

```text
500 folders
Current runtime = 2h 20m
SLA = 2h
```

Test:

```text
Concurrency = 20 → 2h 20m
Concurrency = 50 → 1h 35m
Concurrency = 100 → 1h 30m
```

If 50 has acceptable cost and resource utilization:

```text
Concurrency = 50
```

may be the better choice.

---

# 45. For Each + Monitoring

Monitor:

```text
Total iterations
Successful iterations
Failed iterations
Retry count
Runtime per iteration
Slowest iterations
Concurrency
Overall Job duration
```

This helps identify:

```text
Transient failure
Persistent failure
Data skew
Resource bottleneck
SLA degradation
```

---

# 46. For Each + Notifications

Do not necessarily alert on every successful iteration.

Instead consider:

```text
Individual retry
→ usually operational noise

Retries exhausted
→ actionable alert

Critical iteration failed
→ alert

Overall For Each failure
→ alert

SLA breach
→ alert
```

The correct notification design depends on operational requirements.

---

# 47. For Each + Repair

A robust architecture:

```text
For Each
   ↓
500 iterations
   ↓
497 SUCCESS
2 SUCCESS after retry
1 FAILED
   ↓
Alert
   ↓
Fix root cause
   ↓
Repair failed work
```

The goal is targeted recovery.

---

# 48. End-to-End Example

## Requirement

Every night:

```text
500 customer folders
```

must be processed.

### Architecture

```text
2 AM Trigger
      ↓
Job Parameters
      ↓
processing_date
      ↓
Discover Folders
      ↓
Task Value
      ↓
For Each
      ↓
Concurrency = 50
      ↓
Process Folder
      ↓
Retry transient failures
      ↓
Successful iterations continue
      ↓
Failed iterations isolated
      ↓
Retries exhausted
      ↓
Alert
      ↓
Targeted repair
```

---

# 49. Professional Decision Framework

When you see a For Each scenario:

```text
1. Is the same logic applied to many inputs?
          ↓
       For Each

2. Are inputs static or dynamically discovered?
          ↓
       Choose appropriate input mechanism

3. Are iterations independent?
          ↓
       Safe parallelism may be possible

4. What concurrency is appropriate?
          ↓
       Measure + consider resources/SLA/cost

5. What happens if one iteration fails?
          ↓
       Isolate if independent

6. Is retry appropriate?
          ↓
       Transient → likely yes
       Permanent → fix first

7. Is retry safe?
          ↓
       Check idempotency

8. What happens after retries are exhausted?
          ↓
       Alert + targeted recovery

9. Does downstream processing require every iteration?
          ↓
       Apply business dependency

10. Is there skew?
          ↓
       Optimize the bottleneck rather than blindly increasing concurrency
```

---

# 50. Exam Decision Rules

### Rule 1

> **Use For Each when the same task logic must process multiple inputs.**

### Rule 2

> **For Each concurrency controls parallel iterations, not Job Runs.**

### Rule 3

> **Default For Each concurrency is 1.**

### Rule 4

> **Higher concurrency is not automatically better.**

### Rule 5

> **Independent iterations can continue even when another iteration fails.**

### Rule 6

> **One failed iteration can still make the overall workflow unsuccessful.**

### Rule 7

> **Retry transient failures; don't blindly retry deterministic failures.**

### Rule 8

> **Retries must be evaluated for idempotency.**

### Rule 9

> **Recover the smallest safe unit of failed work.**

### Rule 10

> **Use Task Values + Dynamic References when For Each input is generated dynamically.**

---

# 51. Common Exam Traps

## Trap 1

> "Increase concurrency to maximum to meet SLA."

Not necessarily.

Correct reasoning:

```text
Profile
→ Test
→ Measure
→ Select appropriate concurrency
```

---

## Trap 2

> "One For Each iteration failed, so all iterations failed."

Incorrect.

Independent iterations can have different outcomes.

---

## Trap 3

> "For Each concurrency = 100 means 100 Job Runs."

Incorrect.

It means:

```text
Up to 100 iterations
```

inside the For Each execution.

---

## Trap 4

> "Retry always solves failure."

Incorrect.

Classify the failure first.

---

## Trap 5

> "MERGE automatically guarantees idempotency."

Incorrect.

The matching condition and write design must be deterministic.

---

## Trap 6

> "If 499/500 succeeded, rerun all 500."

Not necessarily.

Prefer targeted recovery if safe.

---

## Trap 7

> "Increasing concurrency solves all performance problems."

Incorrect.

Potential bottlenecks include:

```text
Source
Target
Network
Compute
Data skew
```

---

# 52. Quick Reference

| Requirement | Solution |
|---|---|
| Same logic over many inputs | For Each |
| Runtime-generated inputs | Task Value + Dynamic Reference |
| Parallel iterations | For Each concurrency |
| One iteration fails | Isolate/retry if appropriate |
| Temporary network failure | Retry/backoff |
| Permanent permission failure | Fix root cause |
| Safe reprocessing | Idempotent design |
| Failed work only | Targeted repair |
| Large input data | Store externally, pass reference |
| SLA issue | Measure + optimize + tune concurrency |
| Uneven workload | Investigate skew/bottleneck |
| Multiple Job Runs | Job concurrency |

---

# 53. Final Mental Model

```text
                  WORK ITEMS
                      │
                      ↓
                  For Each
                      │
             ┌────────┼────────┐
             ↓        ↓        ↓
           Item 1   Item 2   Item 3
             │        │        │
             └────────┼────────┘
                      ↓
               Controlled
                Concurrency
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
      SUCCESS                  FAILURE
                                  ↓
                              Classify
                                  ↓
                    ┌─────────────┴─────────────┐
                    ↓                           ↓
                Transient                  Permanent
                    ↓                           ↓
              Retry/Backoff               Fix root cause
                    ↓                           ↓
                 Success                     Repair
                    │                           │
                    └─────────────┬─────────────┘
                                  ↓
                              Monitoring
```

---

# 54. One-Line Rules to Memorize

```text
For Each
→ Same logic over many inputs.

For Each concurrency
→ Number of iterations allowed to execute concurrently.

Job concurrency
→ Number of Job Runs allowed to overlap.

Independent iteration
→ One iteration can fail without necessarily stopping others.

Retry
→ For potentially transient failures.

Backoff
→ Prevent retry storms.

Idempotency
→ Reprocessing produces the same logical result.

Repair
→ Recover failed work without unnecessary reprocessing.

Task Value
→ Runtime-generated value.

Dynamic Reference
→ Access runtime value/metadata.

SLA optimization
→ Measure first, then tune concurrency.

Data skew
→ Parallelism alone may not solve uneven work.
```

---

# 55. Official Documentation

For certification preparation, verify current behavior against Databricks documentation:

- [For Each task](https://docs.databricks.com/aws/en/jobs/tasks/for-each)
- [Job parameters](https://docs.databricks.com/aws/en/jobs/job-parameters)
- [Dynamic value references](https://docs.databricks.com/aws/en/jobs/dynamic-value-references)
- [Task values](https://docs.databricks.com/aws/en/dev-tools/databricks-utils)
- [Task configuration and retries](https://docs.databricks.com/aws/en/jobs/configure-task)
- [Run If conditions](https://docs.databricks.com/aws/en/jobs/run-if)

---
