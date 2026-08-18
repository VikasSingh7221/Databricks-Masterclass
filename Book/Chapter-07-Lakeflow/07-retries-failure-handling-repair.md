# 07. Retries, Failure Handling & Repair

> **Databricks Data Engineer Professional — Certification Notes**
>
> **Source of truth:** Current official Databricks documentation.
>
> **Important:** Retry and repair behavior can differ between normal Jobs and Continuous Jobs. Always identify the Job type before answering a scenario.

---

# 1. Why Failure Handling Matters

A production workflow must answer:

```text
What happens when a task fails?
```

A robust design considers:

```text
Failure
   ↓
Identify cause
   ↓
Transient or permanent?
   ↓
Retry?
   ↓
Retry-safe?
   ↓
Still failing?
   ↓
Repair / recovery
   ↓
Monitor / notify
```

The objective is not simply:

> "Retry everything."

The objective is:

> **Recover safely while preserving correctness and avoiding unnecessary reprocessing.**

---

# 2. Task Failure

A task can fail for many reasons:

```text
Network failure
Temporary service outage
Permission problem
Invalid SQL
Bad configuration
Data-quality failure
Source unavailable
Target unavailable
Timeout
Application error
```

The correct response depends on the cause.

---

# 3. Transient vs Permanent Failure

## Transient Failure

A transient failure may disappear without changing the application/configuration.

Examples:

```text
Temporary network timeout
Temporary service unavailable
Transient infrastructure issue
Temporary connection failure
```

Potential response:

```text
Retry
```

---

## Permanent Failure

A permanent/deterministic failure is unlikely to succeed if you simply execute the same operation again.

Examples:

```text
Permission denied
Invalid SQL
Missing object
Invalid configuration
Incorrect path
Schema incompatibility
```

Potential response:

```text
Fix root cause
      ↓
Retry / repair
```

---

# 4. Retry Policy

Lakeflow Jobs allows you to configure retry behavior for failed task runs.

Current Databricks documentation states that for most normal Job configurations, the default is:

```text
No task retries
```

unless another configuration such as serverless auto-optimization or continuous-job behavior applies.

Official documentation:

https://docs.databricks.com/aws/en/jobs/configure-task

---

# 5. What Does a Retry Do?

Suppose:

```text
Task A
```

fails.

With a retry policy:

```text
Attempt 1
   ↓
FAILED
   ↓
Retry
   ↓
Attempt 2
   ↓
SUCCESS
```

The task is executed again.

The retry does not mean:

```text
Entire Job starts from scratch
```

It is a retry of the failed task run.

---

# 6. Retry Count

A retry policy specifies how many times a failed task can be retried.

Conceptually:

```text
Initial attempt
      +
Configured retries
```

Example:

```text
Retries = 3
```

means the task can be retried according to that configured policy after its initial failed attempt.

Always read the wording carefully:

```text
Retry count
```

is not necessarily the same as:

```text
Total number of executions
```

---

# 7. Retry Interval

A retry policy can specify an interval between failure and the next retry.

Current Databricks documentation describes the retry interval as the time between the start of the failed run and the subsequent retry run. :contentReference[oaicite:1]{index=1}

Conceptually:

```text
Attempt 1
   ↓
FAILED
   ↓
Wait
   ↓
Attempt 2
```

---

# 8. Retry Backoff

Backoff means increasing the delay between retry attempts.

Conceptually:

```text
Attempt 1 → FAIL
     ↓
Wait 10 sec

Attempt 2 → FAIL
     ↓
Wait 30 sec

Attempt 3 → FAIL
     ↓
Wait 90 sec
```

The purpose is to avoid continuously hammering an unavailable dependency.

---

# 9. Retry Is Not a Root-Cause Fix

Consider:

```text
Task
 ↓
Permission denied
 ↓
Retry
 ↓
Permission denied
 ↓
Retry
 ↓
Permission denied
```

The retry doesn't solve the permission problem.

Better:

```text
Permission denied
      ↓
Fix permission
      ↓
Verify
      ↓
Retry / Repair
```

### Professional rule

> **Retry transient failures; fix deterministic failures first.**

---

# 10. Retry Safety

Before retrying a failed task, ask:

```text
Did the failed attempt write data?
```

If yes:

```text
Could the retry duplicate or corrupt data?
```

This is the **idempotency** question.

---

# 11. Non-Idempotent Example

Suppose:

```sql
INSERT INTO target
SELECT *
FROM source;
```

The task:

```text
writes 5,000 records
```

then fails.

A retry may execute the entire operation again.

Potential result:

```text
5,000 original records
+
5,000 duplicate records
```

if the target design does not prevent duplication.

---

# 12. Idempotent Recovery

A retry-safe workflow should be designed so repeated processing does not create incorrect results.

Possible strategies include:

```text
MERGE
Delete + Insert
Deterministic overwrite
Deduplication
Checkpointing
```

The appropriate method depends on the data model.

---

# 13. MERGE Does Not Automatically Mean Idempotent

This is an important Professional trap.

Incorrect:

> "We use MERGE, so retries are automatically safe."

Correct:

> **The MERGE condition and overall write design must be deterministic and retry-safe.**

Example:

```text
Stable business key
       ↓
Deterministic match
       ↓
MERGE
```

is much safer than using an unstable value as the matching identity.

---

# 14. Retry + Timeout

Current Databricks documentation states:

> If both Timeout and Retries are configured, the timeout applies to **each retry**.

Example:

```text
Timeout = 30 minutes
Retries = 2
```

Conceptually:

```text
Attempt 1
→ maximum 30 min

Retry 1
→ maximum 30 min

Retry 2
→ maximum 30 min
```

Therefore, don't interpret the timeout as necessarily applying once to the entire sequence.

Official documentation:

https://docs.databricks.com/aws/en/jobs/configure-task

---

# 15. Timeout vs Retry

These solve different problems.

### Timeout

Answers:

> **How long can this task attempt run?**

### Retry

Answers:

> **Should the failed attempt be executed again?**

Example:

```text
Task
 ↓
Runs too long
 ↓
Timeout
 ↓
Failure
 ↓
Retry policy
 ↓
Retry
```

---

# 16. Timeout Does Not Mean Retry

A task timing out does not inherently mean:

```text
Retry automatically
```

Retry behavior depends on the configured retry policy and Job type.

---

# 17. Retry Strategy

A good retry strategy considers:

```text
Failure type
Retry count
Retry interval
Timeout
Backoff
Idempotency
External-system capacity
SLA
Cost
```

Don't choose:

```text
Maximum retries
```

just because more retries appear safer.

---

# 18. Retry Storm

Suppose:

```text
100 tasks fail
```

and all retry immediately.

```text
100 failures
      ↓
100 retries
      ↓
External service still unavailable
      ↓
100 more failures
```

This can make an outage worse.

Controlled retry intervals/backoff help avoid this pattern.

---

# 19. Retry and SLA

Suppose:

```text
SLA = 2 hours
```

and:

```text
Initial attempt = 60 min
Retry = 60 min
```

A retry strategy could potentially consume most or all of the SLA.

Therefore:

> **Retry policy must be designed together with SLA requirements.**

Consider:

```text
Expected failure frequency
Retry count
Retry delay
Task duration
Remaining SLA
```

---

# 20. Retry and Cost

Retries consume compute and can increase cost.

Example:

```text
Task normally = $1
```

If it executes:

```text
1 initial attempt
+
3 retries
```

the workload can consume substantially more resources.

Therefore:

> **Retries are a reliability mechanism, but they also have performance and cost implications.**

---

# 21. When NOT to Retry

Avoid blindly retrying:

```text
Invalid SQL
Permission denied
Invalid configuration
Missing required table
Invalid credentials
Incorrect code
Deterministic data validation failure
```

Instead:

```text
Identify cause
   ↓
Fix cause
   ↓
Repair/re-run
```

---

# 22. When Retry Makes Sense

Good candidates include:

```text
Temporary network error
Temporary service outage
Transient infrastructure issue
Temporary connection issue
```

provided the operation is retry-safe.

---

# 23. Retry + For Each

Suppose:

```text
For Each
   ↓
500 folders
```

and:

```text
Folder 127 → FAILED
```

while:

```text
Other folders → SUCCESS
```

If the failed iteration has a retry policy:

```text
Folder 127
   ↓
Retry
   ↓
SUCCESS
```

Other independent iterations don't need to be rerun merely because Folder 127 failed.

---

# 24. Retry + Idempotency

Consider:

```text
Folder 127
   ↓
Partial write
   ↓
Failure
```

A retry can cause duplication if the write isn't safe.

Therefore:

```text
Failure
   ↓
Retry?
   ↓
Is operation idempotent?
```

must be part of the design.

---

# 25. Retry Exhaustion

Suppose:

```text
Initial attempt → FAILED
Retry 1 → FAILED
Retry 2 → FAILED
Retry limit reached
```

The task remains unsuccessful.

At this point:

```text
Alert
+
Root-cause investigation
+
Repair/recovery
```

may be appropriate.

---

# 26. Repair Run

Lakeflow Jobs supports repairing failed or canceled Job Runs.

Current Databricks documentation describes repair runs as a way to rerun unsuccessful tasks and their dependent tasks as appropriate.

Official documentation:

https://docs.databricks.com/aws/en/jobs/repair-job-failures

---

# 27. Why Repair Instead of Full Rerun?

Suppose:

```text
A → B → C → D
```

and:

```text
A = SUCCESS
B = SUCCESS
C = FAILED
D = SKIPPED
```

A full rerun would unnecessarily repeat:

```text
A
B
```

A repair run can target the unsuccessful portion and dependent work.

Conceptually:

```text
A ✅
B ✅
C ❌
D skipped

Repair
   ↓
C
   ↓
D
```

This reduces unnecessary processing.

---

# 28. Repair Is Not the Same as Retry

This distinction is important.

## Retry

Usually:

```text
Task fails
   ↓
Configured automatic retry
```

It is part of the task's failure-handling policy.

---

## Repair

Usually:

```text
Job Run fails
   ↓
Fix problem
   ↓
Operator initiates Repair Run
```

It is a targeted recovery mechanism.

### Mental model

```text
Retry
→ automatic failure recovery

Repair
→ targeted operational recovery
```

---

# 29. Repair After Fixing Root Cause

Suppose:

```text
Folder 127
   ↓
Permission denied
```

Do not simply keep retrying.

Better:

```text
Permission denied
       ↓
Fix permission
       ↓
Verify access
       ↓
Repair failed work
```

This is a strong Professional answer.

---

# 30. Repair and Parameters

Current Databricks documentation allows parameters to be added or edited for tasks being repaired.

Repair-time parameter values can override existing values.

This is useful when a failed run needs a controlled correction.

Example:

```text
Original:
processing_date = 2026-08-17

Repair:
processing_date = 2026-08-16
```

Use this carefully and only when the business/recovery requirement supports it.

---

# 31. Repair and Downstream Tasks

Suppose:

```text
A → B → C → D
```

and:

```text
B = FAILED
C = SKIPPED
D = SKIPPED
```

After repairing B:

```text
B → SUCCESS
```

dependent downstream work may also need to be rerun.

The repair mechanism identifies unsuccessful tasks and dependent tasks that need to be rerun.

---

# 32. Repair and Successful Tasks

The goal of a repair run is to avoid unnecessarily repeating already successful work.

Example:

```text
A → SUCCESS
B → SUCCESS
C → FAILED
```

You generally don't want:

```text
A → rerun
B → rerun
C → rerun
```

when only C needs recovery.

---

# 33. Repair Safety

Before repairing:

```text
1. Identify root cause
2. Fix root cause
3. Verify the fix
4. Check idempotency
5. Check whether partial writes occurred
6. Check downstream impact
7. Repair the smallest safe unit
```

---

# 34. Repair vs Full Rerun

### Prefer Repair when:

```text
Most work succeeded
Failure is isolated
Root cause is fixed
Recovery is retry-safe
```

### Full rerun may be appropriate when:

```text
The whole run is invalid
Outputs are intentionally rebuilt
The pipeline is designed for full replacement
A global issue affected all outputs
```

The choice depends on the workflow design.

---

# 35. Failure Handling Architecture

A mature workflow:

```text
                    Task
                     ↓
                  Failure
                     ↓
               Identify cause
                     ↓
              ┌──────┴──────┐
              ↓             ↓
          Transient      Permanent
              ↓             ↓
            Retry        Fix cause
              ↓             ↓
        Retry succeeds?   Repair
          /       \
        YES        NO
         ↓          ↓
      Continue    Alert
                    ↓
                  Repair
```

---

# 36. Failure Isolation

Suppose:

```text
A ──→ B

C ──→ D
```

If:

```text
A = FAILED
```

then:

```text
B
```

may be affected.

But:

```text
C → D
```

is independent.

Therefore:

```text
A/B branch → affected

C/D branch → can continue
```

Do not unnecessarily stop unrelated branches.

---

# 37. Failure vs Skipped/Excluded

Do not assume:

```text
FAILED = SKIPPED = EXCLUDED
```

These are different states.

A downstream task may not run because:

```text
Upstream failure
```

or:

```text
Run If condition not satisfied
```

or:

```text
Task disabled
```

The recovery strategy depends on why the task did not execute.

---

# 38. Current Job Run Outcomes

Current Databricks documentation identifies Job Run outcomes including:

```text
Succeeded
Succeeded with failures
Failed
Skipped
Timed Out
Canceled
```

A particularly important distinction:

> A Job Run can be **Succeeded with failures** when some tasks failed but all leaf tasks were successful.

Official documentation:

https://docs.databricks.com/aws/en/jobs/monitor

---

# 39. Leaf Tasks and Job Status

Databricks determines Job Run status based on the outcomes of the Job's **leaf tasks**.

A leaf task is:

> A task with no downstream dependencies.

Conceptually:

```text
A → B → C
```

C is the leaf task.

If:

```text
C = FAILED
```

the Job Run can be failed.

But if a non-leaf task fails while downstream leaf tasks ultimately succeed, the Job Run can potentially be:

```text
Succeeded with failures
```

This is an important Professional-level detail.

---

# 40. Succeeded With Failures

Example:

```text
A ──→ B
```

Suppose:

```text
A = FAILED
B = SUCCESS
```

If B is the leaf task and the workflow's conditions allow it to execute successfully, the overall Job Run may be:

```text
Succeeded with failures
```

rather than simply:

```text
Failed
```

### Important

Do not determine Job status by looking only at:

```text
"Did any task fail?"
```

You must understand the leaf-task model.

---

# 41. Failure Handling and Run If

Consider:

```text
Validation
   │
   ├── SUCCESS → Transformation
   │
   └── FAILED → Alert
```

If Validation fails:

```text
Transformation
→ not executed

Alert
→ executes
```

The workflow can intentionally handle failure rather than simply terminating every branch.

---

# 42. Failure Handling and Business Criticality

Suppose:

```text
Validation = 99.5% passed
```

but one critical category fails.

The correct response may be:

```text
Critical Transformation
→ BLOCK
```

while:

```text
Independent Audit
→ CONTINUE
```

The best answer is determined by business impact and dependencies.

---

# 43. Data Validation Failure

A validation failure is not always an infrastructure failure.

Example:

```text
Row count mismatch
Expected = 1,000,000
Actual = 999,500
```

The task may execute successfully technically but produce:

```text
Validation = FAIL
```

Don't automatically retry infrastructure-level execution.

Instead:

```text
Investigate data discrepancy
      ↓
Determine business threshold
      ↓
Block/allow downstream processing
```

---

# 44. Retry vs Validation Failure

Suppose:

```text
Validation:
Expected = 1,000,000
Actual = 999,500
```

Retrying immediately may produce the same result.

If the mismatch is caused by:

```text
Late-arriving data
```

then the correct response might be:

```text
Wait / incremental lookback / targeted backfill
```

rather than blind retry.

---

# 45. Retry vs Late-Arriving Data

Suppose:

```text
99% data arrives within 3 days
```

and:

```text
1% arrives later
```

A validation failure due to missing late data may require:

```text
Lookback
+
Targeted backfill
```

rather than:

```text
Retry 5 times
```

### Professional principle

> **Retry is for execution failures; data-timing problems may require a processing strategy.**

---

# 46. Failure Recovery and Idempotency

A robust architecture:

```text
Task
 ↓
Failure
 ↓
Partial output?
 ↓
Idempotent?
 ↓
Retry / Repair
```

If the answer to:

```text
"Can I safely execute this again?"
```

is:

```text
NO
```

then recovery needs a more careful strategy.

---

# 47. Failure Recovery Decision Tree

Use this in scenario questions:

```text
Task failed
    ↓
What caused it?
    ↓
┌───────────────┴───────────────┐
↓                               ↓
Transient                    Permanent
↓                               ↓
Is retry safe?                Fix cause
↓                               ↓
YES / NO                      Verify fix
↓                               ↓
Retry                       Repair/re-run
↓
Success?
 /    \
YES     NO
↓       ↓
Continue Alert + recovery
```

---

# 48. Retry + Compute

Retries consume compute.

If:

```text
Task = expensive Spark transformation
```

and:

```text
Retries = 5
```

a persistent failure can waste significant compute.

Therefore:

```text
Retry count
+
Failure probability
+
Task cost
+
SLA
```

should be considered together.

---

# 49. Retry + External Systems

Suppose the task writes to:

```text
External API
```

A retry may send the same request twice.

This can cause:

```text
Duplicate API request
Duplicate transaction
Duplicate record
```

Therefore, API workflows may need:

```text
Idempotency key
```

or another safe-write strategy.

---

# 50. Retry + Database Writes

For database writes, consider:

```text
Transaction boundary
Commit status
Partial write
Unique keys
MERGE
Deduplication
```

before enabling aggressive retries.

---

# 51. Continuous Jobs — Special Case

Continuous Jobs have different retry behavior.

Current Databricks documentation states:

```text
Continuous Job
→ automatic exponential-backoff failure handling
```

You cannot configure normal retry policies in a continuous Job.

Instead, continuous Jobs automatically retry the entire Job on failure using exponential backoff.

Official documentation:

https://docs.databricks.com/aws/en/jobs/continuous

---

# 52. Continuous Job Retry

Conceptually:

```text
Continuous Job
     ↓
Failure
     ↓
Exponential backoff
     ↓
Restart
     ↓
Failure
     ↓
Longer backoff
     ↓
Restart
```

There is no fixed finite limit on continuous Job retries at the Job level.

The system increases the retry period up to its maximum and continues retrying.

---

# 53. Continuous Job Task Retry Mode

Current documentation allows a continuous Job to use:

```text
Task retry mode = On failure
```

or:

```text
Task retry mode = Never
```

The default is documented as:

```text
On failure
```

This is different from normal Job retry configuration.

---

# 54. Continuous Job Exam Trap

Do not apply normal Job retry rules blindly to continuous Jobs.

Remember:

```text
Normal Job
→ configurable task retry policy

Continuous Job
→ exponential backoff at Job level
→ optional task retry mode
```

---

# 55. Repair vs Continuous Retry

For continuous Jobs:

```text
Failure
 ↓
Automatic exponential backoff
 ↓
New run
```

This is different from manually repairing a normal failed Job Run.

If the continuous Job enters an exponential backoff state, Databricks provides a way to restart the run and reset the retry period.

---

# 56. Failure Handling and Notifications

A mature workflow should distinguish:

```text
Initial failure
```

from:

```text
Retries exhausted
```

Often the second is more actionable.

Example:

```text
Task fails
 ↓
Retry 1
 ↓
Retry 2
 ↓
Success
```

Potentially no critical alert is necessary.

But:

```text
Task fails
 ↓
Retries exhausted
 ↓
Job remains unsuccessful
```

should generally generate an operational alert according to the team's notification policy.

---

# 57. Failure Handling and Monitoring

Monitor:

```text
Failure count
Retry count
Retry duration
Failure reason
Affected task
Affected iteration
Repair history
SLA impact
```

This allows teams to distinguish:

```text
Occasional transient failure
```

from:

```text
Systemic production problem
```

---

# 58. Repair Run Example

Suppose:

```text
Extract       → SUCCESS
Validate      → SUCCESS
Transform     → FAILED
Load Gold     → SKIPPED
Audit         → SUCCESS
```

Root cause:

```text
Permission issue in Transform
```

Fix:

```text
Permission corrected
```

Repair:

```text
Transform
   ↓
Load Gold
```

Do not unnecessarily rerun:

```text
Extract
Validate
Audit
```

if they don't need to be rerun.

---

# 59. Repair Run With Parameters

Suppose the failed task needs a corrected value.

Original:

```text
target_schema = gold_old
```

Repair:

```text
target_schema = gold
```

Repair-time parameter values can override the existing values for the repaired tasks.

This should be used deliberately because the repair is now executing with different inputs.

---

# 60. Failure Recovery Checklist

Before recovery:

```text
□ Identify failed task
□ Identify root cause
□ Determine transient/permanent
□ Check partial writes
□ Check idempotency
□ Fix root cause if required
□ Determine affected downstream tasks
□ Check SLA impact
□ Choose retry vs repair
□ Validate output after recovery
□ Notify stakeholders if required
```

---

# 61. Professional Scenario: Permission Failure

### Scenario

```text
Folder 127
→ Permission denied
```

Other folders succeeded.

Best reasoning:

```text
Permission failure
      ↓
Permanent/deterministic
      ↓
Fix permission
      ↓
Verify access
      ↓
Repair/retry Folder 127
```

Not:

```text
Increase retries to 10
```

---

# 62. Professional Scenario: Network Timeout

### Scenario

```text
Folder 127
→ Temporary network timeout
```

Write operation is idempotent.

Best reasoning:

```text
Transient
   ↓
Retry
   ↓
Backoff if appropriate
```

If retry succeeds:

```text
Continue
```

---

# 63. Professional Scenario: Partial Write

### Scenario

```text
Task wrote 50%
then failed
```

Retry blindly?

```text
NO
```

First determine:

```text
Can the write be safely repeated?
```

If not:

```text
Clean up / reconcile
or
use a safe recovery strategy
```

Then retry/repair.

---

# 64. Professional Scenario: 99.5% Success

Suppose:

```text
1,000,000 records
999,500 passed
500 mismatched
```

Don't automatically:

```text
Retry
```

First ask:

```text
What is the business threshold?
Are the 500 records critical?
Is the mismatch transient?
Does downstream processing depend on those records?
```

Potential outcomes:

```text
Warning + continue
```

or:

```text
Block affected downstream processing
```

depending on business requirements.

---

# 65. Professional Scenario: Independent Branch

Architecture:

```text
Customer Validation
        ↓
Customer Gold

Audit Source
        ↓
Audit Report
```

Customer Validation fails.

If Audit Report is independent:

```text
Customer Gold → BLOCK
Audit Report   → CONTINUE
```

Don't unnecessarily stop the independent branch.

---

# 66. Professional Scenario: Failed Non-Leaf Task

Architecture:

```text
A → B → C
```

Suppose:

```text
B = FAILED
C = SUCCESS
```

If the workflow's configured control flow allows C to run and C is a successful leaf task, the Job Run can be:

```text
Succeeded with failures
```

This is why you must understand leaf-task-based Job Run status.

---

# 67. Professional Decision Framework

When a question asks:

> "What should you do after a task fails?"

Think:

```text
1. What failed?

2. Why did it fail?

3. Is the failure transient?

4. Is retry appropriate?

5. Is retry safe?

6. Did the failed attempt partially write?

7. Will retry violate SLA/cost constraints?

8. If retry fails, what needs repair?

9. Which downstream tasks are affected?

10. Can independent branches continue?

11. Does business validation require blocking?

12. What notification is actionable?
```

---

# 68. Exam Decision Rules

### Rule 1

> **Do not retry deterministic failures blindly.**

### Rule 2

> **Retry is appropriate for potentially transient failures when the operation is retry-safe.**

### Rule 3

> **Always consider idempotency before retrying writes.**

### Rule 4

> **Timeout and retry are different controls.**

### Rule 5

> **When both timeout and retries are configured, the timeout applies to each retry.**

### Rule 6

> **Repair is targeted recovery after a failed/canceled Job Run.**

### Rule 7

> **Fix the underlying issue before repairing a permanent failure.**

### Rule 8

> **Avoid rerunning successful work unnecessarily.**

### Rule 9

> **Independent branches can often continue when another branch fails.**

### Rule 10

> **Continuous Jobs have specialized exponential-backoff failure handling.**

### Rule 11

> **Job Run status is determined using leaf-task outcomes, so a Job can be `Succeeded with failures`.**

---

# 69. Common Exam Traps

## Trap 1

> Permission denied → increase retries.

❌ Wrong.

Fix the permission issue first.

---

## Trap 2

> Network timeout → manually rerun the entire Job.

❌ Not necessarily.

Use configured retry when appropriate and retry-safe.

---

## Trap 3

> Failed task → rerun all successful tasks.

❌ Usually unnecessary.

Use targeted recovery.

---

## Trap 4

> Retry = repair.

❌ Different mechanisms.

```text
Retry
→ automatic task-level recovery

Repair
→ targeted recovery of an unsuccessful Job Run
```

---

## Trap 5

> MERGE automatically guarantees idempotency.

❌ Wrong.

The matching/write logic must be deterministic.

---

## Trap 6

> Validation mismatch → retry.

❌ Not automatically.

First determine:

```text
Business threshold
Data timing
Criticality
Root cause
```

---

## Trap 7

> Any failed task means Job Run = Failed.

❌ Not always.

Current Job Run status considers leaf-task outcomes and can be:

```text
Succeeded with failures
```

---

## Trap 8

> Continuous Job uses normal retry configuration.

❌ Not exactly.

Continuous Jobs have specialized exponential-backoff failure handling.

---

# 70. Quick Reference

| Situation | Appropriate response |
|---|---|
| Temporary network failure | Retry |
| Temporary service outage | Retry |
| Permission denied | Fix permission, then recover |
| Invalid SQL | Fix SQL, then recover |
| Partial write | Check idempotency/reconcile before retry |
| Retry-safe operation | Automatic retry can be appropriate |
| Retries exhausted | Alert + investigate + repair |
| Failed Job Run | Repair after fixing cause |
| Independent branch | Can continue if prerequisites are satisfied |
| Data validation mismatch | Investigate business/data condition |
| Historical correction | Backfill/recovery strategy |
| Continuous Job failure | Exponential backoff |
| Timeout | Limits each attempt when retries are configured |

---

# 71. Final Mental Model

```text
                    TASK FAILURE
                         │
                         ↓
                  Identify Cause
                         │
                ┌────────┴────────┐
                ↓                 ↓
            Transient          Permanent
                │                 │
                ↓                 ↓
          Retry appropriate?   Fix root cause
                │                 │
          ┌─────┴─────┐           ↓
          ↓           ↓        Verify fix
         YES          NO           │
          ↓           ↓            ↓
       Retry       Investigate   Repair
          │
      Retry-safe?
       /       \
     YES        NO
      ↓          ↓
    Retry      Safe recovery
      │
   Success?
    /    \
  YES     NO
   ↓       ↓
Continue  Alert
           ↓
         Repair
```

---

# 72. One-Line Rules to Memorize

```text
Retry
→ Automatic recovery for configured task failures.

Permanent failure
→ Fix the cause before recovery.

Transient failure
→ Retry may be appropriate.

Retry safety
→ Check idempotency and partial writes.

Timeout
→ Maximum duration of an attempt.

Repair
→ Targeted recovery of failed/canceled Job Run work.

Full rerun
→ Use only when the workflow/business requirement requires it.

Failure isolation
→ Don't block unrelated branches.

Validation failure
→ Treat as a data/business decision, not automatically an infrastructure retry.

Continuous Job
→ Exponential-backoff failure handling.

Leaf task
→ A task with no downstream dependencies.

Succeeded with failures
→ Some tasks failed, but all leaf tasks succeeded.
```

---

# 73. Official Documentation

Use these as the primary references for this chapter:

- [Configure and edit tasks — retries and timeouts](https://docs.databricks.com/aws/en/jobs/configure-task)
- [Troubleshoot and repair job failures](https://docs.databricks.com/aws/en/jobs/repair-job-failures)
- [Configure and edit Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/configure-job)
- [Monitor Lakeflow Jobs — Job Run status](https://docs.databricks.com/aws/en/jobs/monitor)
- [Run jobs continuously — failure handling](https://docs.databricks.com/aws/en/jobs/continuous)
- [Configure task dependencies](https://docs.databricks.com/aws/en/jobs/run-if)

---

## Chapter Status

**07. Retries, Failure Handling & Repair — COMPLETE ✅**

### Key verified facts

```text
Normal task retries
→ Configurable retry policy

Default
→ Most normal configurations do not retry failed tasks

Timeout + Retry
→ Timeout applies to each retry

Repair
→ Can repair failed/canceled Job Runs

Repair
→ Reruns unsuccessful tasks and relevant dependent tasks

Continuous Job
→ Exponential-backoff failure handling

Continuous Job
→ No normal retry policies

Continuous Job
→ Optional task retry mode

Job Run status
→ Based on leaf-task outcomes

Possible outcome
→ Succeeded with failures
```
