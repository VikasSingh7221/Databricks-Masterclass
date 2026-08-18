# 13. Final One-Page Revision Sheet

> **Databricks Data Engineer Professional — Lakeflow Jobs**
>
> This is the final rapid-revision sheet.
>
> Use it after completing Chapters 01–12.
>
> The goal is:
>
> ```text
> Recognize scenario
>      ↓
> Identify requirement
>      ↓
> Select feature
>      ↓
> Apply correct behavior
>      ↓
> Avoid trap
> ```

---

# 1. THE MASTER MODEL

```text
                 LAKEFLOW JOB
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      TRIGGER      TASKS        PARAMETERS
        │             │             │
        ↓             ↓             ↓
       WHEN         WHAT          INPUT
                      │
                      ↓
                 DEPENDENCIES
                      │
                      ↓
                  RUN IF
                      │
                      ↓
                  FOR EACH
                      │
                      ↓
                 CONCURRENCY
                      │
                      ↓
                 RETRY / REPAIR
                      │
                      ↓
              MONITOR / ALERT
                      │
                      ↓
                PRODUCTION
```

---

# 2. THE MOST IMPORTANT DISTINCTIONS

Memorize these:

```text
Trigger
→ WHEN does the Job start?

Dependency
→ WHICH task depends on which?

Run If
→ UNDER WHAT upstream condition does a task run?

Parameter
→ INPUT / CONFIGURATION

Task Value
→ RUNTIME-GENERATED OUTPUT

For Each
→ ITERATE over inputs

For Each Concurrency
→ HOW MANY iterations run in parallel?

Job Concurrency
→ HOW MANY Job Runs overlap?

Retry
→ AUTOMATIC recovery attempt

Repair
→ TARGETED recovery of unsuccessful work

Notification
→ ALERT

Monitoring
→ OBSERVE / ANALYZE
```

---

# 3. TRIGGERS

## Scheduled

```text
Run at 2 AM every day
→ Scheduled Trigger
```

## File Arrival

```text
Run when files arrive
→ File Arrival Trigger
```

## Table Update

```text
Run when table changes
→ Table Update Trigger
```

## Continuous

```text
Continuously running workload
→ Continuous
```

### Remember

```text
Trigger
≠
Dependency
```

Trigger starts the Job.

---

# 4. FILE ARRIVAL

If files arrive in bursts:

```text
Wait after last change
→ wait for arrival burst to settle
```

If the requirement is:

```text
Don't trigger too frequently
```

think:

```text
Minimum time between triggers
```

### Important

```text
File Arrival
→ determines WHEN

Processing logic
→ determines WHAT
```

---

# 5. TASK DEPENDENCIES

Example:

```text
A → B → C
```

Means:

```text
B depends on A
C depends on B
```

If:

```text
A fails
```

then dependent downstream work should not blindly proceed.

But independent branches can continue.

---

# 6. INDEPENDENT BRANCHES

```text
A → B

C → D
```

If:

```text
A fails
```

then:

```text
B → affected

C → D → independent
```

### Golden rule

> **Don't block work that doesn't actually depend on the failed work.**

---

# 7. RUN IF — CORE CONDITIONS

| Condition | Meaning |
|---|---|
| `ALL_SUCCEEDED` | All upstream succeeded |
| `AT_LEAST_ONE_SUCCEEDED` | At least one upstream succeeded |
| `AT_LEAST_ONE_FAILED` | At least one upstream failed |
| `ALL_FAILED` | All upstream failed |
| `NONE_FAILED` | No upstream failed |
| `ALL_DONE` | Upstream tasks completed |

Always evaluate the actual task states.

---

# 8. SKIPPED IS NOT FAILED

Critical trap:

```text
SKIPPED ≠ FAILED
```

Example:

```text
A = SUCCESS
B = SKIPPED
```

Then:

```text
ALL_SUCCEEDED
→ NOT satisfied
```

Don't convert:

```text
SKIPPED
```

into:

```text
FAILED
```

---

# 9. RUN IF EXAMPLES

### Example 1

```text
A = SUCCESS
B = SUCCESS
```

```text
ALL_SUCCEEDED
→ TRUE
```

### Example 2

```text
A = SUCCESS
B = FAILED
```

```text
AT_LEAST_ONE_FAILED
→ TRUE
```

### Example 3

```text
A = FAILED
B = SUCCESS
```

```text
AT_LEAST_ONE_SUCCEEDED
→ TRUE
```

### Example 4

```text
A = FAILED
B = SKIPPED
```

```text
ALL_FAILED
→ FALSE
```

because:

```text
SKIPPED ≠ FAILED
```

---

# 10. PARAMETERS

Use Job Parameters for:

```text
environment
processing_date
region
load_type
source
```

Think:

```text
Configuration
```

---

# 11. TASK VALUES

Use Task Values when:

```text
Task A generates a runtime value
        ↓
Task B needs that value
```

Mental model:

```text
Job Parameter
→ INPUT

Task Value
→ OUTPUT
```

---

# 12. PARAMETER TRAP

Do not assume parameter precedence.

If a question gives conflicting values:

```text
Job parameter
Task-specific parameter
Dynamic reference
Task value
```

identify:

```text
Where did the value originate?
Which task consumes it?
Which documented precedence applies?
```

Do not guess.

---

# 13. PARAMETERS ARE NOT SECRETS

Never use normal Job parameters for:

```text
Password
API key
Access token
Private key
```

Use appropriate:

```text
Secret management
Authentication
Permissions
```

---

# 14. FOR EACH

Use For Each when:

```text
Many independent inputs
```

Example:

```text
500 folders
```

Architecture:

```text
For Each
 ├── Folder 1
 ├── Folder 2
 ├── Folder 3
 └── ...
```

---

# 15. FOR EACH CONCURRENCY

Example:

```text
500 inputs
Concurrency = 50
```

means:

```text
500 total iterations
Up to 50 concurrently
```

Not:

```text
50 total
```

and not:

```text
500 concurrent
```

---

# 16. JOB CONCURRENCY

```text
Max concurrent runs = 1
```

means:

```text
Only one run of that Job at a time
```

This is different from:

```text
For Each concurrency
```

---

# 17. CONCURRENCY

Remember:

```text
Job concurrency
→ Job Runs

For Each concurrency
→ Iterations

Workspace concurrency
→ Overall task-run capacity
```

---

# 18. CONCURRENCY MULTIPLIER

Example:

```text
Job concurrency = 5

For Each concurrency = 50
```

Potentially:

```text
5 × 50
=
250
```

concurrent iteration tasks.

Then check:

```text
Workspace
Source
Target
Compute
Cost
```

---

# 19. MAXIMUM CONCURRENCY TRAP

Never think:

```text
Maximum concurrency
→ Best performance
```

Instead:

```text
Benchmark
 ↓
Find SLA point
 ↓
Check capacity
 ↓
Check cost
 ↓
Choose appropriate level
```

---

# 20. SLA DECISION

Example:

```text
SLA = 2 hours
```

Testing:

```text
10  → 3h
25  → 2h 15m
50  → 1h 45m
100 → 1h 35m
```

Preferred:

```text
50
```

if:

```text
50 reliably meets SLA
+
100 provides little benefit
+
50 costs less
```

---

# 21. COMPUTE

For supported Job workloads:

```text
Prefer Serverless
```

For workloads not supported by serverless:

```text
Use supported Classic compute
```

For production classic Jobs:

```text
Jobs compute
```

is preferred over:

```text
All-purpose compute
```

---

# 22. SERVERLESS TRAP

Never memorize:

```text
Serverless supports everything
```

Incorrect.

Some workloads require:

```text
Classic Jobs compute
```

Always consider task type and current supported capabilities.

---

# 23. SHARED COMPUTE

Shared compute can help reduce:

```text
Startup overhead
```

but it is not automatically best.

Use separate compute when tasks have:

```text
Different resource needs
Different libraries
Different configurations
Isolation requirements
```

---

# 24. PERFORMANCE

If a Job is slow:

```text
DON'T immediately increase workers.
```

Check:

```text
CPU
Memory
Shuffle
Data skew
I/O
Small files
Source
Target
```

Then optimize the actual bottleneck.

---

# 25. COST

Correct principle:

```text
Lowest acceptable cost
while satisfying
Correctness + Reliability + SLA
```

Not:

```text
Cheapest possible configuration
```

---

# 26. PERFORMANCE vs COST

Example:

```text
A = $5, 3 hours

B = $8, 1.5 hours
```

SLA:

```text
2 hours
```

Then:

```text
A → unacceptable

B → acceptable
```

---

# 27. RETRY

Retry is appropriate mainly for:

```text
Transient failures
```

Examples:

```text
Temporary network failure
Temporary service unavailability
Transient connection problem
```

---

# 28. PERMANENT FAILURE

Examples:

```text
Permission denied
Invalid SQL
Missing table
Wrong configuration
Invalid path
```

Correct:

```text
Fix root cause
 ↓
Verify
 ↓
Retry/Repair
```

Not:

```text
Retry repeatedly
```

---

# 29. RETRY SAFETY

Before retrying a write:

```text
Could the previous attempt have partially succeeded?
```

If yes:

```text
Is the write idempotent?
```

If no:

```text
Blind retry is risky
```

---

# 30. IDEMPOTENCY

A retry-safe operation should produce the same correct final state when executed again.

Possible patterns:

```text
MERGE with correct stable keys
Deterministic overwrite
Delete + insert for controlled partition
Deduplication
```

But:

```text
MERGE
≠
automatically idempotent
```

---

# 31. RETRY vs REPAIR

```text
Retry
→ Automatic configured recovery

Repair
→ Targeted recovery of unsuccessful Job Run
```

---

# 32. REPAIR

If:

```text
499 tasks succeeded
1 failed
```

and:

```text
Root cause fixed
```

prefer:

```text
Targeted repair
```

when safe.

Avoid unnecessarily rerunning successful work.

---

# 33. RECOVERY FLOW

```text
Failure
 ↓
Find root cause
 ↓
Fix root cause
 ↓
Check retry safety
 ↓
Repair/retry
 ↓
Validate
 ↓
Continue downstream
```

---

# 34. VALIDATION

Technical success:

```text
Task = SUCCESS
```

does not guarantee:

```text
Data = CORRECT
```

Always distinguish:

```text
Execution correctness
+
Data correctness
```

---

# 35. BUSINESS CRITICALITY

Example:

```text
99.9% valid
```

but:

```text
1 critical financial record invalid
```

Potentially:

```text
BLOCK
```

Business criticality can require a stricter rule than a simple percentage threshold.

---

# 36. VALIDATION FAILURE

If:

```text
Transaction validation = FAILED
```

and:

```text
Customer validation = SUCCESS
```

with independent branches:

```text
Customer Gold
→ CONTINUE

Transaction Gold
→ BLOCK
```

---

# 37. BLAST RADIUS

Golden rule:

> **Block the smallest downstream scope required for correctness.**

Don't do:

```text
One branch failed
 ↓
Entire Job stops
```

unless the business dependency actually requires it.

---

# 38. CDC

Remember:

```text
CDC
→ WHAT changed

Validation
→ WHETHER result is acceptable/correct
```

CDC does not replace validation.

---

# 39. INCREMENTAL

Normal processing:

```text
Incremental
```

Historical correction:

```text
Targeted Backfill
```

Good architecture:

```text
Normal path
→ Incremental

Exception path
→ Backfill
```

---

# 40. TRIGGER + CONCURRENCY

Suppose:

```text
Schedule = every 15 minutes
Runtime = 45 minutes
```

Ask:

```text
Can runs overlap safely?
```

If yes:

```text
Controlled concurrency
```

may be appropriate.

If no:

```text
Limit concurrent runs
```

and consider queueing.

---

# 41. QUEUEING

Remember:

```text
Concurrency
→ How many can run simultaneously?

Queueing
→ What happens when another run cannot start?
```

If runs are required:

```text
Queue
```

may be preferable to skipping, subject to the applicable Job configuration and limits.

---

# 42. MONITORING

Use:

```text
Jobs UI
→ Current execution

Run details
→ Task status / logs

Timeline
→ Performance

System tables
→ Historical analysis

Billing/system tables
→ Cost
```

---

# 43. NOTIFICATIONS

Use actionable notifications.

Examples:

```text
Job failure
SLA breach
Retries exhausted
Critical validation failure
Important operational issue
```

Avoid unnecessary alert noise.

---

# 44. DURATION WARNING

Requirement:

> Alert if Job exceeds 2 hours but allow it to continue.

Answer:

```text
Duration Warning
```

Not:

```text
Timeout
```

because:

```text
Warning
→ Alert

Timeout
→ Terminates execution
```

---

# 45. JOB STATUS

Don't infer the entire Job status from:

```text
One failed task
```

Consider:

```text
DAG
Task outcomes
Run If
Dependencies
Leaf tasks
```

A Job can have failed tasks while its overall outcome is not simply interpreted as "failed" in every case.

---

# 46. PRODUCTION IDENTITY

Production Job:

```text
Service Principal
```

Preferred over:

```text
Developer personal identity
```

because production should not depend on one individual's account.

---

# 47. LEAST PRIVILEGE

Give only required permissions.

Example:

```text
READ:
source

WRITE:
target
```

Don't automatically give:

```text
ADMIN
```

---

# 48. UNITY CATALOG

Production architecture should use:

```text
Unity Catalog
```

for governed:

```text
Data
Permissions
Access
Lineage/Governance
```

Use supported compute/access modes.

---

# 49. PRODUCTION ARCHITECTURE

Strong production pattern:

```text
Trigger
 ↓
Ingestion
 ↓
Validation
 ↓
Transformation
 ↓
Audit
 ↓
Monitoring
 ↓
Notification
 ↓
Recovery
```

with:

```text
Service Principal
Unity Catalog
Parameterization
Idempotency
Controlled concurrency
```

---

# 50. DEPENDENCY DESIGN

Good:

```text
            Ingestion
                ↓
            Validation
                ↓
        ┌───────┴───────┐
        ↓               ↓
    Customer         Transaction
      Gold              Gold
        │               │
        └───────┬───────┘
                ↓
              Audit
```

Only create dependencies that represent real requirements.

---

# 51. ARTIFICIAL DEPENDENCY TRAP

Bad:

```text
A → B → C → D
```

when:

```text
B, C, D
```

are independent.

Artificial dependencies create:

```text
Longer runtime
Larger blast radius
Less parallelism
```

---

# 52. ONE GIANT NOTEBOOK TRAP

Bad:

```text
One Notebook
 ├── Ingestion
 ├── Validation
 ├── Transformation
 ├── Audit
 └── Notification
```

Problems:

```text
Poor visibility
Large blast radius
Difficult recovery
Difficult testing
```

Prefer meaningful task boundaries.

---

# 53. TOO MANY TASKS TRAP

Don't create:

```text
Hundreds of tiny tasks
```

without meaningful boundaries.

Balance:

```text
Observability
+
Recovery
+
Maintainability
```

against:

```text
DAG complexity
```

---

# 54. EVENT-DRIVEN ARCHITECTURE

Requirement:

```text
React when data arrives
```

Think:

```text
Event/File Arrival
```

not:

```text
Constant polling
```

when the event-driven trigger is supported and appropriate.

---

# 55. EVENT-DRIVEN ≠ EXACTLY ONCE

Even event-driven workflows can experience:

```text
Repeated events
Overlapping runs
Retries
Partial failures
```

Therefore:

```text
Idempotency
+
Concurrency control
```

may still be necessary.

---

# 56. PRODUCTION STREAMING

Current Databricks production guidance favors:

```text
Lakeflow Jobs
+
Continuous scheduling
+
Jobs compute
```

for production Structured Streaming workloads, subject to the current documented configuration and limitations.

Do not automatically assume:

```text
All-purpose compute
```

is the production recommendation.

---

# 57. STREAMING TRIGGER TRAP

Don't confuse:

```text
Continuous Job scheduling
```

with:

```text
Structured Streaming trigger interval
```

They are different layers.

---

# 58. THE PROFESSIONAL DECISION ORDER

When reading a scenario:

```text
1. BUSINESS REQUIREMENT
        ↓
2. DATA CORRECTNESS
        ↓
3. DEPENDENCIES
        ↓
4. CONTROL FLOW
        ↓
5. FAILURE BEHAVIOR
        ↓
6. RECOVERY SAFETY
        ↓
7. CONCURRENCY
        ↓
8. SLA
        ↓
9. COST
        ↓
10. MONITORING
```

---

# 59. 10-SECOND EXAM METHOD

When time is limited:

```text
WHAT starts it?
→ Trigger

WHAT depends on what?
→ Dependency

WHAT condition?
→ Run If

HOW many inputs?
→ For Each

HOW much parallelism?
→ Concurrency

WHAT if it fails?
→ Retry / Repair

IS retry safe?
→ Idempotency

WHAT must continue?
→ Independent branches

WHAT is the SLA?
→ Performance

WHAT is the cost?
→ Optimization
```

---

# 60. WORDS THAT SHOULD TRIGGER YOUR ATTENTION

When you see these words:

```text
Critical
Independent
Transient
Permanent
Retry-safe
Idempotent
SLA
Cost
Overlap
Batch
Incremental
Historical
Backfill
Production
Business threshold
Maximum
Minimum
```

slow down and analyze the constraint.

---

# 61. WORDS THAT OFTEN APPEAR IN DISTRACTORS

Be careful with:

```text
Always
Never
Maximum
All
Entire Job
Retry everything
Stop everything
Full reload
Increase concurrency
Use admin
Use personal account
```

They are not automatically wrong, but they should trigger scrutiny.

---

# 62. ELIMINATION RULES

Eliminate an answer if it:

```text
❌ Violates correctness

❌ Ignores a dependency

❌ Retries permanent failures

❌ Reruns unnecessary work

❌ Blocks independent work

❌ Maximizes concurrency without evidence

❌ Ignores SLA

❌ Ignores cost

❌ Uses personal identity for production

❌ Treats parameters as secrets

❌ Treats skipped as failed

❌ Treats technical success as data correctness
```

---

# 63. THE 20 RULES TO MEMORIZE

```text
1. Trigger = WHEN.

2. Dependency = ORDER.

3. Run If = CONDITION.

4. Parameter = INPUT.

5. Task Value = RUNTIME OUTPUT.

6. For Each = ITERATION.

7. For Each concurrency = PARALLEL ITERATIONS.

8. Job concurrency = PARALLEL JOB RUNS.

9. Skipped ≠ Failed.

10. Technical success ≠ Data correctness.

11. CDC ≠ Validation.

12. Retry = appropriate automatic recovery.

13. Repair = targeted recovery.

14. Fix permanent failures before recovery.

15. Retry writes only when recovery is safe.

16. Block affected downstream scope, not unrelated work.

17. More concurrency ≠ automatically better.

18. Cheapest ≠ automatically best.

19. Service Principal → production identity.

20. Correctness + SLA + Reliability come before optimization.
```

---

# 64. MASTER ARCHITECTURE

```text
                     TRIGGER
                        ↓
                  LAKEFLOW JOB
                        ↓
                   PARAMETERS
                        ↓
                    INGESTION
                        ↓
                   VALIDATION
                        ↓
              ┌─────────┴─────────┐
              ↓                   ↓
        Customer Gold       Transaction Gold
              ↓                   ↓
              └─────────┬─────────┘
                        ↓
                      AUDIT
                        ↓
                   MONITORING
                        ↓
                  NOTIFICATION
                        ↓
                 REPAIR/BACKFILL
```

Production controls:

```text
Service Principal
Unity Catalog
Serverless / Jobs Compute
Controlled Concurrency
Retries
Idempotency
SLA Monitoring
Cost Monitoring
```

---

# 65. FINAL EXAM QUESTION FORMULA

For every scenario, mentally ask:

```text
What is the requirement?

What must happen?

What must NOT happen?

What can run independently?

What happens if something fails?

Can I retry safely?

Do I need targeted repair?

How much parallelism is actually required?

What is the SLA?

What is the cost impact?

What is the smallest safe solution?
```

---

# 66. FINAL ONE-LINE FORMULA

```text
BEST ANSWER
=
Correctness
+
Requirement Fit
+
Safe Recovery
+
Appropriate Parallelism
+
SLA
+
Acceptable Cost
-
Unnecessary Complexity
-
Unnecessary Blast Radius
```

---

# 67. LAST-MINUTE REVISION

If you have only 5 minutes before the exam:

```text
Trigger
→ WHEN

Dependency
→ ORDER

Run If
→ CONDITION

Parameter
→ INPUT

Task Value
→ RUNTIME OUTPUT

For Each
→ ITERATION

Concurrency
→ PARALLELISM

Retry
→ TRANSIENT

Repair
→ TARGETED RECOVERY

Idempotency
→ SAFE REPEAT

Validation
→ DATA CORRECTNESS

CDC
→ CHANGES

Skipped
→ NOT FAILED

Production
→ SERVICE PRINCIPAL

Classic production compute
→ JOBS COMPUTE

Supported Job workload
→ SERVERLESS

SLA
→ MINIMUM ACCEPTABLE PERFORMANCE

Cost
→ LOWEST ACCEPTABLE COST

Failure
→ FIX ROOT CAUSE FIRST

Architecture
→ MINIMIZE BLAST RADIUS
```

---