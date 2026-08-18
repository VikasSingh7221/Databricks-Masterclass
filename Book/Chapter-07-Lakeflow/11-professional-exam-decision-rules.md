# 11. Professional Exam Decision Rules

> **Databricks Data Engineer Professional — Certification Notes**
>
> This chapter is the **decision-making framework** for Lakeflow Jobs scenario questions.
>
> Professional questions usually test:
>
> ```text
> Requirement
>     ↓
> Constraint
>     ↓
> Correct Databricks feature
>     ↓
> Correct configuration
>     ↓
> Operational consequence
> ```
>
> Do not answer based only on:
>
> > "Which feature is mentioned?"
>
> Instead ask:
>
> > **"Which solution satisfies the complete requirement with the fewest unnecessary side effects?"**

---

# 1. The Professional Exam Mindset

Associate-level thinking often looks like:

```text
What feature does this question mention?
```

Professional-level thinking should be:

```text
What is the business requirement?
        ↓
What must be guaranteed?
        ↓
What can fail?
        ↓
What must continue?
        ↓
What must be blocked?
        ↓
What happens during retry?
        ↓
What happens during recovery?
        ↓
What happens at scale?
        ↓
What is the safest architecture?
```

---

# 2. The Golden Rule

> **Choose the solution that satisfies the requirement while minimizing unnecessary processing, risk, cost, and blast radius.**

Think:

```text
Correctness
    ↓
Reliability
    ↓
Performance
    ↓
Cost
    ↓
Operational simplicity
```

Do not sacrifice correctness merely to improve cost or speed.

---

# 3. First Question: What Is the Actual Requirement?

Before looking at answer choices, identify:

```text
WHAT?
WHEN?
WHY?
HOW MUCH?
WHAT IF IT FAILS?
WHAT MUST CONTINUE?
WHAT MUST STOP?
```

Example:

> "Process files as soon as they arrive."

Important word:

```text
arrive
```

This suggests:

```text
File Arrival Trigger
```

not:

```text
Scheduled Trigger
```

---

# 4. Second Question: Is the Requirement About WHEN or HOW?

This is one of the most important distinctions.

## WHEN

Use:

```text
Trigger
```

Examples:

```text
Every day at 2 AM
→ Scheduled

When files arrive
→ File Arrival

When table updates
→ Table Update
```

---

## HOW

Use:

```text
Task
Dependencies
Run If
For Each
Parameters
Concurrency
```

Example:

```text
Process 500 folders
```

This is not a trigger problem.

It is:

```text
For Each
+
Concurrency
```

---

# 5. Trigger vs Dependency

Remember:

```text
Trigger
→ Starts the Job

Dependency
→ Controls task execution order
```

Example:

```text
2 AM
 ↓
Job
 ↓
A → B → C
```

Changing:

```text
2 AM → 3 AM
```

does not change:

```text
A → B → C
```

---

# 6. Trigger vs Run If

```text
Trigger
→ When does Job start?

Run If
→ Should this task execute based on upstream outcomes?
```

Example:

```text
Job starts
   ↓
Validation
   ↓
Run If
   ↓
Gold
```

The Run If condition does not determine when the Job itself starts.

---

# 7. Requirement → Feature Mapping

Memorize this table:

| Requirement | Think |
|---|---|
| Run at 2 AM | Scheduled trigger |
| Run when files arrive | File Arrival |
| Run when table changes | Table Update |
| Always-on streaming | Continuous |
| Process many inputs | For Each |
| Control iteration parallelism | For Each concurrency |
| Control Job Run overlap | Max concurrent runs |
| Run based on upstream state | Run If |
| Choose between branches | If/else |
| Pass reusable configuration | Job parameters |
| Pass generated runtime values | Task values / dynamic references |
| Retry transient task failure | Retry |
| Recover failed Job Run | Repair |
| Alert operator | Notifications |
| Analyze historical execution | System tables |
| Protect critical downstream work | Validation + dependencies |
| Historical correction | Targeted backfill |

---

# 8. Data Correctness Comes First

Suppose:

```text
99.5% records passed
500 records mismatched
```

Do NOT automatically conclude:

```text
99.5% > threshold
→ Continue
```

Ask:

```text
Are the 500 records business-critical?
What caused the mismatch?
Is the threshold approved?
Is downstream processing safe?
```

---

# 9. Business Criticality Beats Simple Percentages

Example:

```text
99.9% records valid
```

but:

```text
1 critical financial transaction incorrect
```

The correct response may be:

```text
BLOCK
```

even though the overall percentage is excellent.

### Professional rule

> **Aggregate quality metrics do not override explicit business-critical requirements.**

---

# 10. Validation vs CDC

Remember:

```text
CDC
→ tells you what changed

Validation
→ tells you whether the resulting data satisfies correctness rules
```

CDC does not prove:

```text
Correctness
Completeness
Business validity
```

Therefore:

```text
CDC
+
Validation
```

may be required.

---

# 11. Technical Success vs Data Success

A task can be:

```text
SUCCESS
```

while data is:

```text
INCORRECT
```

Example:

```text
SQL executed successfully
```

but:

```text
500 records missing
```

Therefore:

```text
Task status
≠
Data quality
```

Professional architecture must consider both.

---

# 12. Blocking Decision

When a critical validation fails:

```text
Validation
   ↓
CRITICAL FAILURE
   ↓
Affected downstream processing
   ↓
BLOCK
```

But don't automatically block unrelated branches.

---

# 13. Smallest Safe Blast Radius

Suppose:

```text
Customer
   ↓
Customer Gold

Transaction
   ↓
Transaction Gold
```

Transaction validation fails.

Do:

```text
Transaction Gold
→ BLOCK
```

while:

```text
Customer Gold
→ CONTINUE
```

if Customer is independent and passed.

### Golden rule

> **Block the smallest downstream scope required for correctness.**

---

# 14. Dependency Reasoning

Suppose:

```text
A → B → C
```

If A fails:

```text
B
```

depends on failed work.

Therefore B should not normally proceed unless the workflow explicitly handles the failure through appropriate control flow.

If:

```text
D
```

is independent:

```text
D
```

may continue.

---

# 15. Independent Branch Rule

Architecture:

```text
A → B

C → D
```

If A fails:

```text
B → affected

C → D → independent
```

Do not unnecessarily stop:

```text
C → D
```

---

# 16. Run If Decision Rules

Memorize the semantic meaning.

```text
ALL_SUCCEEDED
→ Every upstream task succeeded.

AT_LEAST_ONE_SUCCEEDED
→ At least one upstream succeeded.

NONE_FAILED
→ No upstream task failed.

ALL_DONE
→ All upstream tasks completed.

AT_LEAST_ONE_FAILED
→ At least one upstream failed.

ALL_FAILED
→ Every upstream task failed.
```

Always evaluate the actual upstream states.

---

# 17. Skipped Is Not Failed

This is a major exam trap.

Suppose:

```text
B = SKIPPED
```

Do not automatically treat it as:

```text
FAILED
```

The condition may behave differently depending on:

```text
FAILED
SKIPPED
EXCLUDED
SUCCESS
```

---

# 18. "At Least One Failed" Rule

Suppose:

```text
A = SUCCESS
B = FAILED
C = SKIPPED
```

Then:

```text
AT_LEAST_ONE_FAILED
```

is:

```text
TRUE
```

because B failed.

---

# 19. "At Least One Succeeded" Rule

Suppose:

```text
A = FAILED
B = SUCCESS
C = SKIPPED
```

Then:

```text
AT_LEAST_ONE_SUCCEEDED
```

is:

```text
TRUE
```

because B succeeded.

---

# 20. ALL_SUCCEEDED Trap

Suppose:

```text
A = SUCCESS
B = SKIPPED
```

Then:

```text
ALL_SUCCEEDED
```

is not satisfied.

Why?

```text
Not all upstream tasks succeeded.
```

---

# 21. ALL_DONE

Suppose:

```text
A = SUCCESS
B = FAILED
C = SKIPPED
```

If all upstream tasks have reached a terminal/completed state appropriate to the condition:

```text
ALL_DONE
```

can be satisfied.

This condition is useful when the downstream task should execute after upstream processing finishes regardless of success/failure.

---

# 22. Dependency vs Run If

Do not confuse:

```text
Dependency
```

with:

```text
Condition
```

Dependency says:

```text
Task B depends on Task A
```

Run If says:

```text
Under what upstream outcome should B execute?
```

---

# 23. Parameter Decision Rule

When a question asks:

> "How should this value be passed to multiple tasks?"

Think:

```text
Job Parameter
```

when the value is known/configuration-level.

Examples:

```text
environment
processing_date
region
load_type
```

---

# 24. Dynamic Runtime Value

If a value is generated by one task:

```text
Task A
 ↓
generates value
 ↓
Task B needs it
```

think:

```text
Task Value
+
Dynamic Value Reference
```

---

# 25. Job Parameter vs Task Value

### Job Parameter

```text
Known/configuration input
```

### Task Value

```text
Runtime-generated value
```

Mental model:

```text
Job Parameter
→ Input into Job

Task Value
→ Output from Task for downstream use
```

---

# 26. Parameter Precedence

Do not rely on memory here.

Always distinguish:

```text
Job parameters
Task parameters
Dynamic references
Task values
```

For supported parameterized tasks, current Databricks documentation defines specific precedence behavior.

### Exam rule

> **When a scenario gives conflicting parameter values, explicitly identify where each value comes from and apply the documented precedence for that task type.**

Do not guess based on variable naming.

---

# 27. Parameters Are Not Secrets

Never choose:

```text
Job parameter = password
```

as a security solution.

Instead use:

```text
Secret management
+
appropriate authentication
```

---

# 28. For Each Decision Rule

If the requirement says:

```text
Process 500 folders independently
```

think:

```text
For Each
```

instead of manually creating:

```text
500 tasks
```

---

# 29. For Each Concurrency

Suppose:

```text
500 inputs
Concurrency = 50
```

Think:

```text
Up to 50 iterations concurrently
```

Not:

```text
500 concurrently
```

---

# 30. For Each Failure Isolation

Suppose:

```text
500 folders
```

and:

```text
Folder 127 = FAILED
```

Other folders may continue depending on the workflow configuration.

Do not assume:

```text
One iteration failed
→ all iterations stop
```

---

# 31. For Each Concurrency Is Not Job Concurrency

```text
Job concurrency
→ Number of Job Runs

For Each concurrency
→ Number of concurrent iterations
```

This distinction appears frequently in scenario questions.

---

# 32. Concurrency Multiplier

Suppose:

```text
Job concurrency = 5
For Each concurrency = 50
```

Potentially:

```text
5 × 50 = 250
```

concurrent iteration tasks.

Then evaluate:

```text
Workspace
Source
Target
Compute
Cost
```

---

# 33. Don't Maximize Concurrency Automatically

Wrong reasoning:

```text
Higher concurrency
→ faster
→ therefore choose maximum
```

Correct:

```text
Benchmark
 ↓
Find point where SLA is met
 ↓
Check cost/capacity
 ↓
Choose appropriate level
```

---

# 34. SLA Rule

Suppose:

```text
SLA = 2 hours
```

Testing:

```text
Concurrency 10 → 3h
Concurrency 25 → 2h 20m
Concurrency 50 → 1h 40m
Concurrency 100 → 1h 35m
```

Choose:

```text
50
```

if:

```text
50 reliably meets SLA
+
100 provides little additional benefit
+
50 costs less
```

---

# 35. Trigger Decision Rule

Ask:

> **What event should start the Job?**

```text
Time
→ Scheduled

Files
→ File Arrival

Table change
→ Table Update

Model lifecycle event
→ Model Update

Always-on workload
→ Continuous
```

---

# 36. File Arrival Debounce

If files arrive in bursts:

```text
file1
file2
file3
...
```

and the requirement is:

> Process the batch after arrivals settle.

Think:

```text
Wait after last change
```

---

# 37. File Arrival Cooldown

If the requirement is:

> Don't trigger more often than every N minutes.

Think:

```text
Minimum time between triggers
```

---

# 38. Cooldown vs Debounce

Memorize:

```text
Wait after last change
→ "Wait until files stop arriving."

Minimum time between triggers
→ "Don't trigger too frequently."
```

---

# 39. Continuous Job Rule

Use Continuous when the requirement is:

```text
Continuously running workload
```

especially appropriate streaming workloads.

But verify:

```text
Task dependency limitations
Compute support
Streaming trigger compatibility
Retry behavior
```

---

# 40. Continuous Job Trap

Current documented behavior:

```text
Continuous Job
→ One running instance
```

and:

```text
Continuous Job
→ Cannot use task dependencies
```

Do not apply normal DAG assumptions blindly.

---

# 41. Retry Decision Rule

When a task fails:

```text
WHY?
```

Then classify:

```text
Transient
```

or:

```text
Permanent
```

---

# 42. Transient Failure

Examples:

```text
Temporary network timeout
Temporary service outage
Transient connection problem
```

Potential answer:

```text
Retry
```

provided the operation is retry-safe.

---

# 43. Permanent Failure

Examples:

```text
Permission denied
Invalid SQL
Missing table
Invalid configuration
Wrong path
```

Correct reasoning:

```text
Fix root cause
 ↓
Verify
 ↓
Retry/Repair
```

---

# 44. Retry Is Not a Root-Cause Fix

Wrong:

```text
Permission denied
 ↓
Retry × 10
```

Correct:

```text
Permission denied
 ↓
Fix permission
 ↓
Repair
```

---

# 45. Retry Safety

Before retrying a write ask:

```text
Could the previous attempt have partially written data?
```

If yes:

```text
Is the operation idempotent?
```

If no:

```text
Blind retry is risky.
```

---

# 46. Idempotency Rule

A retry-safe operation should produce the correct final state even when executed more than once.

Examples of strategies:

```text
MERGE with stable keys
Deterministic overwrite
Delete + insert for a controlled partition
Deduplication
```

The exact strategy depends on the workload.

---

# 47. MERGE Trap

Wrong:

> "MERGE means the operation is automatically idempotent."

Correct:

> **The matching condition and write logic must make repeated execution safe.**

---

# 48. Timeout vs Retry

```text
Timeout
→ How long can one attempt run?

Retry
→ Should another attempt occur?
```

If both are configured:

```text
Timeout applies to each retry attempt
```

---

# 49. Retry vs Repair

```text
Retry
→ Automatic configured task recovery

Repair
→ Targeted recovery of an unsuccessful Job Run
```

---

# 50. Repair Decision

Use Repair when:

```text
Run failed/canceled
+
root cause is fixed
+
targeted recovery is safe
```

Example:

```text
Folder 127 failed
```

Fix:

```text
Permission
```

Then:

```text
Repair failed work
```

instead of unnecessarily rebuilding everything.

---

# 51. Full Rerun vs Repair

Prefer targeted repair when:

```text
Most work succeeded
Failure is isolated
Root cause is fixed
Recovery is safe
```

Full rerun may be appropriate when:

```text
Whole output is invalid
Full rebuild is intentional
Workflow is designed for full replacement
```

---

# 52. Monitoring Decision Rule

Ask:

> **What do I need to know?**

```text
Current execution
→ Jobs UI

Task failure
→ Run details/logs

Slow task
→ Timeline

Historical analysis
→ system.lakeflow

Cost
→ Billing + system tables
```

---

# 53. Notification Decision Rule

Ask:

> **What event should notify someone?**

```text
Job started
→ Start

Job succeeded
→ Success

Job failed
→ Failure

Job slower than expected
→ Duration Warning

Streaming lag
→ Streaming Backlog
```

---

# 54. Retry Notification Trap

If requirement is:

> "Notify after every failed retry attempt."

Think:

```text
Task-level notification
```

not only Job-level failure notification.

---

# 55. Succeeded With Failures Trap

If:

```text
Some tasks failed
```

but:

```text
All leaf tasks succeeded
```

Job Run may be:

```text
Succeeded with failures
```

For notification selection, current Databricks behavior treats this as a successful state.

Therefore:

```text
Success notification
```

is relevant.

---

# 56. Duration Warning vs Timeout

Requirement:

> "Alert if the Job exceeds 2 hours but allow it to continue."

Answer:

```text
Duration Warning
```

Not:

```text
Timeout
```

Because:

```text
Warning
→ notify

Timeout
→ terminate
```

---

# 57. Monitoring Retention

Current documented values:

```text
Jobs UI
→ 60 days

Lakeflow system tables
→ 365 days free retention
```

Use system tables when longer historical analysis is required.

---

# 58. Compute Decision Rule

First ask:

```text
Is serverless supported?
```

If yes:

```text
Is it appropriate for the workload?
```

If yes:

```text
Prefer serverless
```

If no:

```text
Use supported classic compute
```

---

# 59. Production Compute Rule

For classic production Jobs:

```text
Jobs compute
```

is preferred over:

```text
All-purpose compute
```

---

# 60. Shared Compute Decision

Share compute when:

```text
Tasks have compatible requirements
+
sharing is operationally beneficial
```

Separate compute when:

```text
Different workload profiles
Different libraries/configurations
Different resource requirements
Isolation needed
```

---

# 61. Cost Decision Rule

Never choose:

```text
Cheapest
```

automatically.

Choose:

```text
Lowest acceptable cost
that satisfies
correctness + reliability + SLA
```

---

# 62. Cost + Performance

Suppose:

```text
Option A = $5, 3 hours
Option B = $8, 1.5 hours
```

SLA:

```text
2 hours
```

Then:

```text
A = unacceptable
B = acceptable
```

Even though B costs more.

---

# 63. Production Identity Rule

For production:

```text
Service Principal
```

is preferred over:

```text
Developer's personal identity
```

because production execution should not depend on one person's account.

---

# 64. Least Privilege Rule

Ask:

> **What is the minimum permission required?**

Avoid:

```text
Job → Admin access
```

when it only needs:

```text
Read source
Write target
```

---

# 65. Parameterization Rule

If values differ by environment:

```text
DEV
QA
PROD
```

do not hard-code them into the workflow.

Use:

```text
Parameters
+
Environment-specific configuration
```

---

# 66. Secret Rule

Never treat:

```text
Job parameter
```

as:

```text
Secret
```

Use proper secret/authentication mechanisms.

---

# 67. Production Architecture Rule

A strong production architecture usually contains:

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
Security
+
Idempotency
+
Parameterization
+
Controlled concurrency
```

---

# 68. Blast Radius Rule

When a component fails:

> **Stop only what depends on the failed component, unless business requirements require a broader stop.**

Example:

```text
Customer Gold
→ Continue

Transaction Gold
→ Block
```

if only Transaction data failed validation.

---

# 69. Dependency Completeness Rule

Ask:

> **Does downstream processing actually depend on this upstream result?**

If yes:

```text
Create dependency.
```

If no:

```text
Keep it independent.
```

Don't add dependencies merely to make the graph look orderly.

---

# 70. Avoid Artificial Dependencies

Bad:

```text
A → B → C → D
```

when all tasks are actually independent.

This creates:

```text
Longer runtime
Larger failure propagation
Less parallelism
```

Better:

```text
A
├── B
├── C
└── D
```

when independence is real.

---

# 71. Critical Path Rule

Optimize tasks on the:

```text
Critical path
```

first.

If:

```text
A → B → C
```

takes:

```text
2 hours
```

while an independent task takes:

```text
2 minutes
```

optimizing the 2-minute task has little impact on total Job runtime.

---

# 72. Event-Driven Rule

If the requirement says:

> "React when data arrives."

Prefer:

```text
Event-driven trigger
```

over:

```text
Frequent polling schedule
```

when the event trigger is supported and appropriate.

---

# 73. Incremental Processing Rule

Event-driven execution does not automatically make processing incremental.

Remember:

```text
Trigger
→ WHEN

Processing logic
→ WHAT
```

A File Arrival trigger can still launch a full-table scan if the task is written that way.

---

# 74. Backfill Rule

Production systems should have:

```text
Normal incremental path
+
Targeted historical recovery path
```

Do not redesign the normal path around rare backfills.

---

# 75. Queueing Decision Rule

If a new run cannot start:

```text
Should it wait?
```

If yes:

```text
Queueing
```

may be appropriate.

If the run is no longer useful:

```text
Skipping
```

may be acceptable.

Business requirement determines the answer.

---

# 76. Queueing vs Concurrency

Do not confuse:

```text
Concurrency
→ How many can run simultaneously?

Queueing
→ What happens when another run cannot start?
```

---

# 77. Workspace Capacity Rule

Even if:

```text
Job concurrency = 100
```

the workload may still be constrained by:

```text
Workspace task-run capacity
Compute
Source
Target
```

A Job setting does not override platform/resource constraints.

---

# 78. Scenario: 500 Independent Folders

Requirement:

```text
500 independent folders
SLA = 2 hours
```

Testing:

```text
Concurrency 10 → 3h
Concurrency 50 → 1h 40m
Concurrency 100 → 1h 35m
```

Best answer:

```text
Concurrency = 50
```

if:

```text
50 meets SLA
+
cost is acceptable
+
source/target can support it
```

---

# 79. Scenario: One Folder Failed

```text
Folder 127 → FAILED
Other folders → SUCCESS
```

Best approach:

```text
Investigate Folder 127
+
Fix root cause
+
Repair/retry Folder 127
```

Don't automatically rerun all 500.

---

# 80. Scenario: Permission Failure

```text
Permission denied
```

Best answer:

```text
Fix permission
 ↓
Verify
 ↓
Repair
```

Not:

```text
Increase retries
```

---

# 81. Scenario: Temporary Network Failure

```text
Network timeout
```

If:

```text
Operation is idempotent
```

then:

```text
Retry
```

may be appropriate.

---

# 82. Scenario: Validation Mismatch

```text
Expected = 1,000,000
Actual = 999,500
```

Do not automatically:

```text
Retry
```

Ask:

```text
Business threshold?
Critical records?
Late-arriving data?
Root cause?
Downstream impact?
```

---

# 83. Scenario: Customer Passes, Transaction Fails

```text
Customer Validation → SUCCESS
Transaction Validation → FAILED
```

If independent:

```text
Customer Gold → CONTINUE
Transaction Gold → BLOCK
```

---

# 84. Scenario: Alert Only After Failure

Requirement:

> "Send one alert if the Job ultimately fails, not for each retry."

Answer:

```text
Job-level Failure notification
```

---

# 85. Scenario: Alert Every Failed Attempt

Requirement:

> "Alert on each failed task attempt."

Answer:

```text
Task-level notification
```

---

# 86. Scenario: SLA Warning

Requirement:

> "Alert when processing exceeds 2 hours but don't stop it."

Answer:

```text
Duration Warning
```

---

# 87. Scenario: Event-Based Trigger

Requirement:

> "Start processing when files arrive."

Answer:

```text
File Arrival Trigger
```

If files arrive in batches:

```text
Wait after last change
```

may be appropriate.

---

# 88. Scenario: Time-Based Trigger

Requirement:

> "Run every day at 2 AM."

Answer:

```text
Scheduled Trigger
```

---

# 89. Scenario: Table-Based Trigger

Requirement:

> "Start when the source table changes."

Answer:

```text
Table Update Trigger
```

---

# 90. Scenario: Production Identity

Requirement:

> "The Job must continue to work if the developer leaves."

Answer:

```text
Service Principal
```

not:

```text
Developer identity
```

---

# 91. Scenario: Production Cost

Requirement:

> "Reduce cost without violating a 2-hour SLA."

Approach:

```text
Measure current cost
 ↓
Benchmark compute/concurrency
 ↓
Find minimum configuration meeting SLA
 ↓
Compare cost
 ↓
Deploy
```

---

# 92. Scenario: Wrong Compute

Requirement:

```text
Production Job
```

Current configuration:

```text
All-purpose compute
```

Preferred direction:

```text
Serverless
```

if supported and appropriate, otherwise:

```text
Jobs compute
```

---

# 93. Scenario: Independent Audit

Architecture:

```text
Customer → Gold

Audit Source → Audit Report
```

Customer fails.

If Audit does not depend on Customer:

```text
Audit Report → Continue
Customer Gold → Block
```

---

# 94. Scenario: One Critical Validation

```text
99.9% records valid
```

but:

```text
critical regulatory record failed
```

Correct reasoning:

```text
Business criticality
→ overrides simple aggregate threshold
```

Potentially:

```text
Block affected downstream
```

---

# 95. Scenario: CDC

Requirement:

> "We have CDC, so can we trust the Gold table?"

Answer:

```text
Not automatically.
```

CDC tells:

```text
What changed
```

Validation tells:

```text
Whether resulting data is acceptable/correct
```

---

# 96. Scenario: Full Rerun vs Repair

If:

```text
499 tasks succeeded
1 task failed
```

and:

```text
root cause fixed
```

prefer:

```text
Targeted repair
```

rather than:

```text
Full Job rerun
```

when recovery is safe.

---

# 97. Scenario: Retry vs Repair

```text
Task fails during execution
→ Retry may automatically recover it.

Job Run has already failed
→ Repair can target unsuccessful work.
```

---

# 98. Scenario: Retry Safety

If:

```text
Task partially writes
```

then:

```text
Do not blindly retry.
```

First establish:

```text
Idempotency
+
safe recovery
```

---

# 99. Scenario: More Workers

If Job is slow:

```text
Do not immediately increase workers.
```

First inspect:

```text
Timeline
CPU
Memory
Shuffle
I/O
Data skew
Source
Target
```

Then change the bottleneck.

---

# 100. The "Best Answer" Pattern

Professional questions often contain several technically valid options.

Choose the answer that:

```text
1. Meets the requirement
2. Preserves correctness
3. Minimizes unnecessary work
4. Limits failure propagation
5. Is operationally recoverable
6. Meets SLA
7. Avoids unnecessary cost
```

---

# 101. The "Least Change" Principle

When recovering a production failure:

> **Change/reprocess the smallest safe scope.**

Example:

```text
500 folders
1 failed
```

Prefer:

```text
1 folder recovery
```

over:

```text
500 folder rerun
```

when safe.

---

# 102. The "Don't Overreact" Principle

A single failure does not automatically mean:

```text
Stop everything.
```

Ask:

```text
What depends on it?
```

---

# 103. The "Don't Underreact" Principle

A high overall success percentage does not automatically mean:

```text
Continue.
```

Ask:

```text
Is the failed data business-critical?
```

---

# 104. The "Evidence Before Scaling" Principle

Never choose:

```text
Concurrency = 100
```

because:

> "More parallelism is better."

Choose based on:

```text
Testing
Measurements
SLA
Cost
Capacity
```

---

# 105. The "Root Cause Before Recovery" Principle

For failures:

```text
Failure
 ↓
Root cause
 ↓
Fix if necessary
 ↓
Recovery
```

Not:

```text
Failure
 ↓
Retry repeatedly
```

---

# 106. The "Correctness Before Cost" Principle

Suppose:

```text
Option A = cheaper
```

but:

```text
Option B = safer/correct
```

For critical production data:

```text
Correctness wins.
```

Then optimize cost within the correct architecture.

---

# 107. The "SLA Is a Constraint" Principle

SLA should be treated as:

```text
Minimum acceptable performance
```

not:

```text
The faster the better
```

If:

```text
50 concurrency = 1h 45m
100 concurrency = 1h 35m
```

and SLA:

```text
2h
```

then 50 may be enough.

---

# 108. The "Business Requirement Beats Product Feature" Principle

Don't choose:

```text
For Each
```

because the question mentions:

```text
500 records
```

Ask:

> Are the records independent inputs that should be iterated?

Similarly:

Don't choose:

```text
File Arrival
```

just because files exist.

Ask:

> Is file arrival actually the event that should start the workflow?

---

# 109. The "Separate Concerns" Principle

Keep these concepts separate:

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
→ AUTOMATIC RECOVERY

Repair
→ TARGETED RECOVERY

Notification
→ ALERT

Monitoring
→ OBSERVATION
```

---

# 110. Master Decision Tree

Use this during the exam:

```text
START
  │
  ↓
What is the requirement?
  │
  ├── WHEN?
  │     ↓
  │   Trigger
  │
  ├── ORDER?
  │     ↓
  │   Dependency
  │
  ├── CONDITION?
  │     ↓
  │   Run If / If-Else
  │
  ├── MANY INPUTS?
  │     ↓
  │   For Each
  │
  ├── VALUE?
  │     ↓
  │   Parameter / Task Value
  │
  ├── FAILURE?
  │     ↓
  │   Retry / Repair
  │
  ├── ALERT?
  │     ↓
  │   Notification
  │
  ├── PERFORMANCE?
  │     ↓
  │   Compute / Concurrency
  │
  ├── HISTORY?
  │     ↓
  │   System Tables
  │
  └── PRODUCTION?
        ↓
   Security + Governance
   Idempotency
   Recovery
   Monitoring
   SLA
   Cost
```

---

# 111. 10-Second Exam Framework

When time is limited:

```text
1. Identify the trigger.
2. Identify dependencies.
3. Identify critical validation.
4. Identify failure scope.
5. Check retry safety.
6. Check concurrency.
7. Check SLA.
8. Check cost.
9. Choose smallest safe solution.
```

---

# 112. Professional Answer Pattern

When explaining your selected answer mentally:

```text
"The requirement is X.

Therefore feature Y is appropriate.

Because constraint Z exists,
configuration A should be used.

This prevents problem B
while preserving requirement C."
```

Example:

> Files arrive in batches and should be processed once the batch settles.

Reasoning:

```text
Requirement
→ Event-driven

Trigger
→ File Arrival

Batch behavior
→ Wait after last change

Additional safety
→ Controlled Job concurrency
```

---

# 113. Eliminate Wrong Answers

When stuck between options, ask:

### Does it violate correctness?

```text
YES → eliminate
```

### Does it unnecessarily rerun work?

```text
YES → usually eliminate
```

### Does it ignore business criticality?

```text
YES → eliminate
```

### Does it blindly increase concurrency?

```text
YES → suspicious
```

### Does it retry a deterministic failure?

```text
YES → eliminate
```

### Does it block independent work?

```text
YES → suspicious
```

### Does it solve a problem the requirement doesn't have?

```text
YES → likely over-engineered
```

---

# 114. Common Professional Exam Distractors

Watch for answers containing:

```text
"Always"
"Never"
"Maximum"
"All"
"Entire Job"
"Retry everything"
"Stop everything"
"Increase concurrency"
"Use full reload"
```

These words are not automatically wrong, but they should trigger deeper analysis.

---

# 115. High-Value Words in Scenario Questions

Look for:

```text
Critical
Independent
Transient
Permanent
Retry-safe
Idempotent
SLA
Cost
Overlapping
Batch
Incremental
Historical
Targeted
Business threshold
Production
Exactly once
```

These words often reveal the intended decision.

---

# 116. Professional Exam Trap: "Exactly Once"

Do not assume:

```text
Lakeflow Job
→ automatically exactly-once business processing
```

Exactly-once behavior depends on:

```text
Source
Processing
Checkpointing
Write semantics
Idempotency
Target
```

Evaluate the complete architecture.

---

# 117. Professional Exam Trap: "Success"

Do not interpret:

```text
SUCCESS
```

as:

```text
Data is correct
```

Technical execution success and data validation are separate.

---

# 118. Professional Exam Trap: "Retry"

Do not interpret:

```text
Retry
```

as:

```text
Safe for every failure
```

Always ask:

```text
Transient?
Idempotent?
Partial write?
SLA?
Cost?
```

---

# 119. Professional Exam Trap: "Concurrency"

Do not interpret:

```text
Concurrency
```

as:

```text
Unlimited scalability
```

Always ask:

```text
Source?
Target?
Compute?
Workspace?
Cost?
Correctness?
```

---

# 120. Professional Exam Trap: "Trigger"

Do not interpret:

```text
Trigger
```

as:

```text
Task ordering
```

Trigger starts the Job.

Dependencies control task relationships.

---

# 121. Professional Exam Trap: "Skipped"

Do not interpret:

```text
SKIPPED
```

as:

```text
FAILED
```

Understand why the task was skipped and how the relevant Run If condition evaluates that state.

---

# 122. Professional Exam Trap: "One Failed Task"

Do not interpret:

```text
One task failed
```

as:

```text
Everything must stop
```

Check:

```text
Dependencies
Run If
Business criticality
Independent branches
Leaf tasks
```

---

# 123. Professional Exam Trap: "99.5%"

Do not interpret:

```text
99.5%
```

as:

```text
Always acceptable
```

Ask:

```text
What are the remaining records?
```

---

# 124. Professional Exam Trap: "CDC"

Do not interpret:

```text
CDC
```

as:

```text
Data quality validation
```

CDC tells:

```text
What changed
```

not:

```text
Whether the result is correct
```

---

# 125. Professional Exam Trap: "Repair"

Do not repair before:

```text
Root cause fixed
```

unless the failure itself is transient and the recovery is otherwise safe.

---

# 126. Professional Exam Trap: "Full Rerun"

Don't rerun successful work merely because one component failed.

Ask:

```text
Can targeted repair recover the affected scope?
```

---

# 127. Master Rules

Memorize these:

```text
1. Correctness before optimization.

2. Business criticality before aggregate percentages.

3. Trigger answers WHEN.

4. Dependency answers ORDER.

5. Run If answers CONDITION.

6. Parameters provide INPUT.

7. Task Values provide runtime OUTPUT.

8. For Each provides ITERATION.

9. Concurrency provides PARALLELISM.

10. Retry handles appropriate transient failures.

11. Repair handles targeted recovery.

12. Idempotency makes recovery safer.

13. Validation checks correctness.

14. CDC identifies changes.

15. Independent branches should remain independent.

16. Block the smallest affected scope.

17. Fix permanent failures before recovery.

18. Don't maximize concurrency blindly.

19. Meet SLA at acceptable cost.

20. Monitor execution and data correctness separately.

21. Use production identities, not personal identities.

22. Never treat parameters as secrets.

23. Prefer targeted recovery over unnecessary full reruns.

24. Use backfills for exceptional historical corrections.

25. Choose the simplest architecture that satisfies all requirements.
```

---

# 128. Final Professional Mental Model

```text
                   BUSINESS REQUIREMENT
                            │
                            ↓
                      DATA CORRECTNESS
                            │
                            ↓
                      DEPENDENCIES
                            │
                            ↓
                        CONTROL FLOW
                            │
                            ↓
                         COMPUTE
                            │
                            ↓
                       CONCURRENCY
                            │
                            ↓
                           SLA
                            │
                            ↓
                          COST
                            │
                            ↓
                        MONITORING
                            │
                            ↓
                       NOTIFICATION
                            │
                            ↓
                      FAILURE RECOVERY
                            │
                            ↓
                       PRODUCTION
```

The Professional exam is often testing whether you can reason through this entire chain rather than identify one feature.

---

# 129. Final Exam Rule

When two answers both appear technically possible:

> **Choose the one that satisfies the stated business requirement while introducing the least unnecessary risk, cost, processing, and failure propagation.**

That is the core Professional-level decision rule.

---

## Chapter Status

**11. Professional Exam Decision Rules — COMPLETE ✅**

### Core exam framework

```text
Requirement
    ↓
Correctness
    ↓
Dependencies
    ↓
Control Flow
    ↓
Compute
    ↓
Concurrency
    ↓
SLA
    ↓
Cost
    ↓
Monitoring
    ↓
Recovery
```
