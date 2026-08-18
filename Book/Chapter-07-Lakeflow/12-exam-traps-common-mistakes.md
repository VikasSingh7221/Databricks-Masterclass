# 12. Exam Traps & Common Mistakes

> **Databricks Data Engineer Professional — Certification Notes**
>
> This chapter is a final defense against common Professional-level mistakes.
>
> The exam often gives multiple technically valid options.
> The correct answer is usually the one that best satisfies the **specific requirement and constraints**.

---

# 1. The Biggest Exam Trap

## Don't answer based on the feature name alone.

Question:

> "500 independent folders must be processed."

Don't immediately choose:

```text
For Each
```

First determine:

```text
Are the folders independent?
Is iteration required?
Is parallelism required?
What is the SLA?
What concurrency is safe?
```

The Professional exam tests architecture, not keyword matching.

---

# 2. Trigger ≠ Dependency

### Trigger

Answers:

```text
WHEN does the Job start?
```

### Dependency

Answers:

```text
HOW are tasks related?
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

# 3. Trigger ≠ Run If

Trigger:

```text
Starts Job
```

Run If:

```text
Controls task execution based on upstream states
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

Do not use a trigger to solve a downstream conditional execution problem.

---

# 4. Dependency ≠ Condition

Dependency:

```text
B depends on A
```

Run If:

```text
B should run based on A's result
```

These work together.

---

# 5. Scheduled Trigger Trap

Question:

> "Run every day at 2 AM."

Correct:

```text
Scheduled Trigger
```

Not:

```text
Task dependency
Run If
For Each
```

---

# 6. File Arrival Trigger Trap

Question:

> "Run whenever a new file arrives."

Correct direction:

```text
File Arrival Trigger
```

Not:

```text
Every 5-minute schedule
```

unless polling is specifically required.

---

# 7. File Arrival Does Not Mean Incremental

A File Arrival trigger only answers:

```text
WHEN
```

It does not automatically determine:

```text
WHAT DATA
```

the task processes.

You still need appropriate processing logic.

---

# 8. File Arrival Batch Trap

Suppose:

```text
100 files
```

arrive within a short period.

If the requirement is:

> Process once after the batch has finished arriving.

Consider:

```text
Wait after last change
```

Do not assume one trigger event means one file.

---

# 9. Trigger Cooldown Trap

Two concepts are different:

```text
Wait after last change
```

vs

```text
Minimum time between triggers
```

### Wait after last change

Think:

```text
Wait for the arrival burst to settle.
```

### Minimum time between triggers

Think:

```text
Prevent triggers from occurring too frequently.
```

---

# 10. Continuous Job Trap

Don't confuse:

```text
Continuous Job scheduling
```

with:

```text
Structured Streaming trigger
```

They are different concepts.

---

# 11. Continuous Job Dependency Trap

Do not assume a Continuous Job behaves exactly like a normal DAG.

Current Databricks documentation places restrictions on task dependencies for continuous Jobs.

Always verify the current documented behavior when a question involves:

```text
Continuous
+
task dependencies
```

---

# 12. Run If: Skipped ≠ Failed

This is one of the highest-value traps.

Suppose:

```text
A = SUCCESS
B = SKIPPED
```

Do not reason:

```text
B did not run
→ B failed
```

Incorrect.

```text
SKIPPED
≠
FAILED
```

The condition must be evaluated using the actual task state.

---

# 13. ALL_SUCCEEDED Trap

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
Not every upstream task succeeded.
```

---

# 14. AT_LEAST_ONE_SUCCEEDED

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

is satisfied.

Because:

```text
B = SUCCESS
```

---

# 15. AT_LEAST_ONE_FAILED

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

is satisfied.

Because:

```text
B = FAILED
```

---

# 16. ALL_FAILED Trap

Suppose:

```text
A = FAILED
B = SKIPPED
```

Do not conclude:

```text
ALL_FAILED = TRUE
```

because:

```text
SKIPPED ≠ FAILED
```

---

# 17. ALL_DONE Trap

`ALL_DONE` is about upstream tasks reaching their completed/terminal state rather than requiring success.

Therefore:

```text
SUCCESS
FAILED
SKIPPED
```

must be distinguished from:

```text
not yet completed
```

---

# 18. "One Failed Task" Trap

Wrong:

```text
One task failed
→ Entire Job must stop
```

Not necessarily.

Ask:

```text
What depends on that task?
```

Independent branches may still execute.

---

# 19. Independent Branch Trap

Architecture:

```text
A → B

C → D
```

If:

```text
A = FAILED
```

then:

```text
B = affected
```

but:

```text
C → D
```

may remain independent.

Do not create unnecessary failure propagation.

---

# 20. Critical Validation Trap

Suppose:

```text
99.9% records valid
```

but:

```text
1 critical regulatory record invalid
```

Do not automatically:

```text
Continue
```

Business criticality can require:

```text
Block
```

---

# 21. Percentage Threshold Trap

Do not assume:

```text
Above threshold = always continue
```

Ask:

```text
What records failed?
Are they critical?
What does the business policy say?
```

---

# 22. Validation ≠ Execution Success

A task can show:

```text
SUCCESS
```

while producing:

```text
Incorrect data
```

Example:

```text
SQL completed
```

but:

```text
Expected rows = 1,000,000
Actual rows = 900,000
```

Technical success does not prove data correctness.

---

# 23. CDC ≠ Data Validation

CDC tells you:

```text
What changed
```

It does not automatically tell you:

```text
Whether the final data is correct
```

Therefore:

```text
CDC
+
Validation
```

may both be necessary.

---

# 24. "Block Everything" Trap

If:

```text
Transaction validation failed
```

don't automatically block:

```text
Customer Gold
Product Gold
Audit
```

Ask which downstream tasks actually depend on Transaction.

---

# 25. Smallest Safe Scope

Preferred reasoning:

```text
Failure
 ↓
Identify affected dependency graph
 ↓
Block only affected work
```

This minimizes:

```text
Blast radius
```

---

# 26. Job Parameter Trap

Job parameters are for:

```text
Configuration/input
```

Examples:

```text
environment
processing_date
region
load_type
```

They are not automatically:

```text
Secrets
```

---

# 27. Parameter ≠ Secret

Never put:

```text
password
API secret
access token
private key
```

into normal Job parameters.

Use appropriate secret/authentication mechanisms.

---

# 28. Task Value Trap

Task Values are useful for:

```text
Runtime-generated values
```

Example:

```text
Task A
 ↓
generates count
 ↓
Task Value
 ↓
Task B consumes it
```

Don't confuse them with ordinary Job parameters.

---

# 29. Job Parameter vs Task Value

Remember:

```text
Job Parameter
→ Input/configuration

Task Value
→ Runtime output from a task
```

---

# 30. Dynamic Value Reference Trap

A dynamic value reference does not magically convert every value into the desired data type.

Always consider:

```text
What is the source value?
How is it represented?
What does the consuming task expect?
```

---

# 31. Parameter Precedence Trap

If a question gives:

```text
Job parameter = prod

Task-specific value = qa
```

do not blindly assume:

```text
qa
```

or:

```text
prod
```

without identifying:

```text
Which parameter source?
Which task?
Which parameterization mechanism?
```

Apply the documented precedence for that specific context.

---

# 32. For Each Trap

For Each means:

```text
Iterate over a collection of inputs
```

It does not mean:

```text
Run everything simultaneously.
```

Concurrency controls how many iterations can execute at once.

---

# 33. For Each Concurrency Trap

Suppose:

```text
500 inputs
Concurrency = 50
```

Correct mental model:

```text
500 total iterations
Up to 50 executing concurrently
```

Not:

```text
50 total inputs
```

and not:

```text
500 concurrent
```

---

# 34. For Each Failure Trap

Suppose:

```text
Folder 127 = FAILED
```

Do not automatically conclude:

```text
All other folders fail.
```

Other iterations may continue according to the workflow's execution behavior.

---

# 35. Job Concurrency ≠ For Each Concurrency

This is extremely important.

```text
Job concurrency
→ concurrent Job Runs

For Each concurrency
→ concurrent iterations
```

---

# 36. Concurrency Multiplier Trap

Suppose:

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

Then ask:

```text
Workspace capacity?
Source capacity?
Target capacity?
Compute?
Cost?
```

---

# 37. Maximum Concurrency Trap

Do not select the maximum simply because:

```text
Maximum = fastest
```

More concurrency can cause:

```text
Contention
Throttling
Higher cost
Failures
Retries
```

---

# 38. SLA Trap

Suppose:

```text
SLA = 2 hours
```

Testing:

```text
Concurrency 25 → 2h 20m
Concurrency 50 → 1h 45m
Concurrency 100 → 1h 40m
```

Don't automatically choose:

```text
100
```

If:

```text
50 already satisfies SLA
```

then 50 may be the better cost/performance choice.

---

# 39. "More Workers" Trap

If a Job is slow:

```text
Don't immediately increase workers.
```

First identify:

```text
CPU?
Memory?
Shuffle?
Skew?
I/O?
Source?
Target?
Small files?
```

---

# 40. Compute Bottleneck Trap

More CPU does not automatically solve:

```text
Memory bottleneck
Network bottleneck
Source throttling
Target throttling
Data skew
```

Optimize the actual bottleneck.

---

# 41. Serverless Trap

Do not memorize:

```text
Serverless = supports everything
```

Incorrect.

Some workloads require classic compute.

---

# 42. Serverless vs Classic Trap

Decision:

```text
Serverless supported?
        ↓
      YES
        ↓
Is it appropriate?
   ┌────┴────┐
  YES       NO
   ↓         ↓
Serverless  Classic
```

---

# 43. Production Compute Trap

For classic production Jobs:

```text
Jobs compute
```

is preferred over:

```text
All-purpose compute
```

Don't choose all-purpose simply because it already exists.

---

# 44. Shared Compute Trap

Shared compute can be useful, but don't assume:

```text
Shared = always cheaper/better
```

Different workloads may need:

```text
Different libraries
Different memory
Different CPU
Different Spark configuration
```

---

# 45. Cost Trap

Do not choose:

```text
Cheapest
```

without checking:

```text
SLA
Reliability
Correctness
Runtime
```

Correct principle:

```text
Lowest acceptable cost
while satisfying requirements
```

---

# 46. DBU Trap

Do not think:

```text
1 DBU = $1
```

DBU is a:

```text
Usage unit
```

Actual pricing depends on the applicable service/SKU/rate and cloud configuration.

---

# 47. Retry Trap

Retry is not a universal failure solution.

Ask:

```text
Is the failure transient?
Is the operation retry-safe?
Could partial data have been written?
```

---

# 48. Permission Failure Trap

Example:

```text
PERMISSION_DENIED
```

Wrong:

```text
Retry 10 times
```

Correct:

```text
Fix permission
 ↓
Verify
 ↓
Repair/retry
```

---

# 49. Invalid SQL Trap

Example:

```text
Syntax error
```

Don't solve with:

```text
More retries
```

This is normally a deterministic failure.

Fix the code.

---

# 50. Temporary Network Failure

Example:

```text
Temporary connection timeout
```

Potentially:

```text
Retry
```

provided the operation is safe to retry.

---

# 51. Retry + Idempotency Trap

Before retrying a write ask:

```text
Could the previous attempt have partially succeeded?
```

If yes:

```text
Can repeated execution safely produce the correct final state?
```

If no:

```text
Blind retry is dangerous.
```

---

# 52. "MERGE = Idempotent" Trap

MERGE is not automatically idempotent.

Idempotency depends on:

```text
Match condition
Source uniqueness
Write logic
Target state
```

---

# 53. Timeout ≠ Retry

```text
Timeout
→ Maximum allowed execution time for an attempt.

Retry
→ Whether another attempt should be made.
```

These solve different problems.

---

# 54. Retry ≠ Repair

```text
Retry
→ Automatic recovery during execution.

Repair
→ Targeted recovery of an unsuccessful Job Run.
```

---

# 55. Full Rerun Trap

Suppose:

```text
499 tasks succeeded
1 task failed
```

Wrong default:

```text
Rerun everything
```

Better:

```text
Fix root cause
+
Targeted repair
```

when safe.

---

# 56. Repair Before Fixing Root Cause

Don't blindly repair:

```text
Permission failure
```

before fixing:

```text
Permission
```

Correct:

```text
Failure
 ↓
Root cause
 ↓
Fix
 ↓
Verify
 ↓
Repair
```

---

# 57. Historical Backfill Trap

Don't redesign the normal daily pipeline to perform:

```text
Full historical processing
```

just because one historical correction is required.

Better:

```text
Normal incremental pipeline
+
Targeted backfill
```

---

# 58. Incremental ≠ Backfill

```text
Incremental
→ normal forward processing

Backfill
→ exceptional historical processing
```

---

# 59. Notification Trap

Don't configure every possible notification.

Ask:

```text
What does the operator actually need to know?
```

Good alerts:

```text
Critical failure
SLA breach
Retries exhausted
Important data-quality failure
```

Avoid excessive noise.

---

# 60. Job Failure vs Task Failure Notification

If requirement:

> Notify only when the final Job fails.

Think:

```text
Job-level notification
```

If requirement:

> Notify when each task fails.

Think:

```text
Task-level notification
```

---

# 61. Duration Warning Trap

Requirement:

> "Alert if runtime exceeds 2 hours but allow the Job to continue."

Correct:

```text
Duration Warning
```

Not:

```text
Timeout
```

because timeout changes execution behavior.

---

# 62. Succeeded With Failures Trap

A Job can have:

```text
Some failed tasks
```

while still ending in a successful outcome under certain DAG conditions.

Do not infer Job status solely from:

```text
"I saw a failed task."
```

Inspect the overall Job Run state and relevant task graph.

---

# 63. Monitoring Retention Trap

Do not assume:

```text
Jobs UI
```

is the right tool for indefinite historical analysis.

Use:

```text
System tables
```

when longer-term analysis is required and supported.

---

# 64. System Table Trap

System tables are useful for:

```text
Historical analysis
Usage
Billing
Job/task activity
```

They are not the same thing as:

```text
Real-time Job execution UI
```

Use the appropriate monitoring surface.

---

# 65. Production Identity Trap

Don't run production workflows under:

```text
Developer's personal account
```

Prefer:

```text
Service Principal
```

This avoids production dependency on an individual identity.

---

# 66. Least Privilege Trap

Don't choose:

```text
Workspace admin
```

when the Job only needs:

```text
Read source
Write target
```

Use the minimum necessary permissions.

---

# 67. Environment Trap

Don't hard-code:

```text
prod
```

throughout notebooks if the same workflow is intended for:

```text
DEV
QA
PROD
```

Use appropriate configuration/parameterization.

---

# 68. Parameterized Environment Trap

A reusable Job can use:

```text
environment = dev
```

or:

```text
environment = prod
```

rather than duplicating all workflow logic.

But don't use parameters as a replacement for:

```text
Security controls
```

---

# 69. Secret Trap

Never place secrets in:

```text
Notebook source
Git
Job parameters
Logs
Task values
```

Use appropriate secret/authentication mechanisms.

---

# 70. Dependency Overengineering Trap

Bad:

```text
A → B → C → D
```

when:

```text
A
B
C
D
```

are actually independent.

This causes:

```text
Unnecessary waiting
Longer critical path
Larger blast radius
```

---

# 71. Missing Dependency Trap

The opposite is also dangerous.

If:

```text
C requires B
```

but no dependency exists:

```text
B
C
```

may execute incorrectly.

Model real dependencies explicitly.

---

# 72. "One Giant Notebook" Trap

Don't assume fewer tasks always means better architecture.

One giant notebook creates:

```text
Large blast radius
Poor visibility
Hard recovery
Difficult testing
```

Prefer meaningful task boundaries.

---

# 73. "Too Many Tasks" Trap

Don't split everything into:

```text
Hundreds of tiny tasks
```

without a reason.

This creates:

```text
Complex DAG
Operational overhead
Difficult maintenance
```

Use meaningful units of work.

---

# 74. Validation Scope Trap

If only:

```text
Transaction
```

failed validation, don't automatically validate/block:

```text
Customer
Product
Audit
```

unless the architecture requires it.

---

# 75. Audit Dependency Trap

Ask:

> "Does Audit actually depend on the failed dataset?"

If:

```text
Audit uses Customer
Customer passed
```

then:

```text
Audit may continue.
```

Don't block it just because another branch failed.

---

# 76. Failure Propagation Trap

Failure should propagate through:

```text
Dependencies
```

not simply through:

```text
"Same Job"
```

Being in the same Job does not automatically mean:

```text
One failure = everything stops.
```

---

# 77. Event Trigger Trap

Event-driven does not mean:

```text
No concurrency concerns.
```

Multiple events may arrive.

Consider:

```text
Maximum concurrent runs
Queueing
Duplicate events
Target safety
```

---

# 78. Scheduled Trigger Overlap Trap

Suppose:

```text
Schedule = every 15 minutes
Runtime = 45 minutes
```

Ask:

```text
Can multiple runs overlap safely?
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

# 79. Queueing Trap

Queueing answers:

```text
What happens when another run cannot start?
```

It does not mean:

```text
Run more tasks concurrently.
```

---

# 80. Queueing vs Concurrency

```text
Concurrency
→ How many can run?

Queueing
→ What happens to additional runs?
```

---

# 81. Queueing vs Skipping

If a run is not allowed to start:

```text
Queueing enabled
→ Wait

Queueing not applicable/disabled
→ Run may be skipped according to the relevant behavior
```

Always inspect the exact Job configuration and current documentation.

---

# 82. Workspace Limit Trap

A Job-level concurrency value does not override workspace/platform limits.

Example:

```text
Job concurrency = 100
```

does not guarantee:

```text
100 runs will execute immediately.
```

Other constraints may apply.

---

# 83. Cost + Retry Trap

Suppose:

```text
Normal run = $5
```

and it retries 3 times.

Potential compute usage:

```text
$5 × multiple attempts
```

Therefore excessive retries can increase cost significantly.

---

# 84. Cost + Concurrency Trap

Increasing concurrency can:

```text
Reduce runtime
```

but also:

```text
Increase compute usage
Increase target pressure
Increase cost
```

Optimize based on measured results.

---

# 85. SLA + Cost Trap

The cheapest option is not necessarily valid.

Example:

```text
$5 → 3 hours
$8 → 1.5 hours
```

SLA:

```text
2 hours
```

Correct:

```text
$8 option
```

if reliability and other constraints are acceptable.

---

# 86. Performance Mode Trap

Don't assume:

```text
Performance-optimized
```

is always best.

If:

```text
Startup latency is unimportant
```

and:

```text
Batch cost matters
```

consider:

```text
Standard
```

for supported serverless workloads.

---

# 87. "Cheapest Serverless Mode" Trap

Don't choose Standard solely because:

```text
It is cheaper.
```

Check:

```text
Startup requirements
SLA
Workload behavior
```

---

# 88. Production Streaming Trap

Don't automatically use:

```text
All-purpose compute
```

for production Structured Streaming.

Current Databricks production guidance favors:

```text
Lakeflow Jobs
+
Jobs compute
+
continuous scheduling
```

for production streaming workloads, with current documented recommendations/constraints.

---

# 89. Streaming Autoscaling Trap

Do not blindly apply:

```text
Autoscaling
```

to Structured Streaming because:

> "More data = more workers."

Current Databricks production guidance has specific recommendations around autoscaling for streaming workloads.

Always verify the current documentation for the exact streaming architecture.

---

# 90. Exactly-Once Trap

Don't assume:

```text
Lakeflow Jobs
→ exactly-once business result
```

Exactly-once behavior depends on the entire pipeline:

```text
Source
Processing
Checkpointing
Write semantics
Target
Idempotency
```

---

# 91. "Task Succeeded" Trap

This:

```text
Task = SUCCESS
```

only establishes execution success according to the task outcome.

It does not automatically prove:

```text
Data quality
Completeness
Business correctness
```

---

# 92. "Job Started" Trap

A trigger starting the Job does not mean:

```text
All tasks will run.
```

After the Job starts:

```text
Dependencies
Run If
Parameters
Task conditions
```

determine task behavior.

---

# 93. "Dependency Controls Start Time" Trap

Dependency controls:

```text
Task ordering
```

not:

```text
Job start time
```

Schedule/event trigger controls Job start.

---

# 94. "Run If Controls Job Start" Trap

Run If applies to:

```text
Task execution
```

not:

```text
Job trigger
```

---

# 95. "For Each Creates Separate Jobs" Trap

For Each creates:

```text
Iterations
```

within the workflow.

It does not mean:

```text
Each input = separate Job Run
```

---

# 96. "For Each Concurrency = Job Concurrency" Trap

Again:

```text
For Each concurrency
→ iterations

Job concurrency
→ Job Runs
```

---

# 97. "Repair Means Rerun Everything" Trap

Repair is intended to provide:

```text
Targeted recovery
```

rather than blindly repeating all successful work.

---

# 98. "Repair Before Root Cause" Trap

Correct:

```text
Fix root cause
 ↓
Repair
```

Not:

```text
Repair
 ↓
Hope
```

---

# 99. "Retry Forever" Trap

Retries should be:

```text
Controlled
```

with:

```text
Maximum attempts
Backoff
Appropriate failure classification
```

Avoid infinite retry loops for permanent failures.

---

# 100. "Alert Everything" Trap

Too many alerts cause:

```text
Alert fatigue
```

Prefer:

```text
Actionable
Business-relevant
Operationally meaningful
```

notifications.

---

# 101. "Block Everything on Validation Failure" Trap

Ask:

```text
What failed?
What depends on it?
Is it critical?
```

Then:

```text
Block affected downstream scope
```

rather than automatically stopping unrelated work.

---

# 102. "Ignore Validation Because Pipeline Is Green" Trap

Green pipeline:

```text
Technical success
```

does not automatically mean:

```text
Business success
```

---

# 103. "Threshold Always Wins" Trap

A threshold is only meaningful within its:

```text
Business context
```

Critical records may require stricter treatment.

---

# 104. "One Failure Means One Job Failure" Trap

The final Job outcome depends on:

```text
DAG structure
Task outcomes
Run If conditions
Leaf tasks
```

Do not infer the Job result from one task alone.

---

# 105. "Skipped = Success" Trap

Also incorrect.

```text
SKIPPED
≠
SUCCESS
```

The downstream condition must explicitly account for the state.

---

# 106. "Skipped = Failed" Trap

Also incorrect.

```text
SKIPPED
≠
FAILED
```

This distinction is essential for Run If questions.

---

# 107. "All Succeeded" Trap

If one upstream is:

```text
SKIPPED
```

then:

```text
ALL_SUCCEEDED
```

is not satisfied.

---

# 108. "At Least One Succeeded" Trap

If:

```text
A = FAILED
B = SUCCESS
```

then:

```text
AT_LEAST_ONE_SUCCEEDED
```

is satisfied.

---

# 109. "At Least One Failed" Trap

If:

```text
A = SUCCESS
B = FAILED
```

then:

```text
AT_LEAST_ONE_FAILED
```

is satisfied.

---

# 110. "None Failed" Trap

If:

```text
A = SUCCESS
B = SKIPPED
```

don't automatically interpret:

```text
B = FAILED
```

The exact semantics of the condition must be applied.

---

# 111. "Maximum" Trap

Whenever you see:

```text
maximum concurrency
maximum retries
maximum workers
```

ask:

> **Why maximum?**

If the requirement only says:

```text
Meet SLA
```

maximum may be unnecessary.

---

# 112. "Minimum Cost" Trap

Whenever you see:

```text
lowest cost
```

ask:

```text
Does it meet SLA?
Does it maintain reliability?
Does it preserve correctness?
```

---

# 113. "Fastest" Trap

Fastest is not automatically best.

Ask:

```text
How much does it cost?
Is the extra speed needed?
Does it increase failure risk?
```

---

# 114. "Full Reload" Trap

Full reload is not automatically safer.

It can create:

```text
Higher cost
Longer runtime
More target pressure
Larger blast radius
```

Use it when the architecture/business requirement actually calls for it.

---

# 115. "Incremental" Trap

Incremental processing is not automatically correct either.

Ask:

```text
Late-arriving data?
Deletes?
Updates?
Historical corrections?
Missed events?
```

A good production architecture often needs:

```text
Incremental
+
Backfill/recovery mechanism
```

---

# 116. "Event-Driven = Exactly Once" Trap

Event-driven execution can still experience:

```text
Multiple events
Repeated events
Overlapping runs
Retries
Partial failures
```

Therefore design:

```text
Idempotency
+
Concurrency control
```

appropriately.

---

# 117. "Production = More Infrastructure" Trap

Professional architecture does not mean:

```text
More clusters
More tasks
More services
More orchestration
```

It means:

```text
Correct
Reliable
Secure
Observable
Recoverable
Cost-effective
```

---

# 118. "External Orchestrator Is Always Better" Trap

If the workflow is primarily Databricks-native:

```text
Lakeflow Jobs
```

may be the simpler choice.

External orchestration is appropriate when:

```text
Cross-platform dependencies
Enterprise orchestration standards
External systems
```

require it.

---

# 119. "One Tool Must Do Everything" Trap

Don't force Lakeflow Jobs to solve:

```text
Data transformation
Data quality
Secret management
Source ingestion
Enterprise orchestration
```

by itself.

A production architecture combines the appropriate components.

---

# 120. Professional Elimination Strategy

When unsure between answers:

### Eliminate answer if it:

```text
❌ Violates correctness
❌ Ignores dependencies
❌ Retries permanent failures
❌ Reruns unnecessary work
❌ Blocks independent branches
❌ Maximizes concurrency without evidence
❌ Ignores SLA
❌ Ignores cost
❌ Uses personal identity for production
❌ Treats parameters as secrets
❌ Assumes skipped = failed
❌ Assumes technical success = data correctness
```

---

# 121. Final 15 High-Value Traps

Memorize these first:

```text
1. Trigger ≠ dependency.

2. Trigger ≠ Run If.

3. Skipped ≠ failed.

4. Technical success ≠ data correctness.

5. CDC ≠ validation.

6. Job concurrency ≠ For Each concurrency.

7. Maximum concurrency ≠ automatically best.

8. Retry ≠ root-cause fix.

9. Retry ≠ repair.

10. Repair ≠ full rerun.

11. Parameter ≠ secret.

12. Independent branch ≠ automatically blocked.

13. Validation threshold ≠ automatic permission to continue.

14. Cheapest ≠ best.

15. More compute ≠ guaranteed performance improvement.
```

---

# 122. Final Mental Model

When you see a Professional scenario:

```text
             READ THE REQUIREMENT
                       ↓
                IDENTIFY THE EVENT
                       ↓
                IDENTIFY DEPENDENCIES
                       ↓
                IDENTIFY DATA CRITICALITY
                       ↓
                IDENTIFY FAILURE TYPE
                       ↓
                 CHECK RETRY SAFETY
                       ↓
                 CHECK CONCURRENCY
                       ↓
                    CHECK SLA
                       ↓
                   CHECK COST
                       ↓
             MINIMIZE BLAST RADIUS
                       ↓
             CHOOSE SIMPLEST SAFE DESIGN
```

---

# 123. Final Rule

> **Don't ask "Which Databricks feature does this question mention?"**

Ask:

> **"What outcome does the business require, what can safely happen in parallel, what must stop if something fails, and what is the smallest reliable solution?"**

That is the Professional-level mindset.

---

## Chapter Status

**12. Exam Traps & Common Mistakes — COMPLETE ✅**

### Highest-priority traps

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

Validation
→ DATA CORRECTNESS

CDC
→ DATA CHANGES

Skipped
→ NOT Failed

Technical Success
→ NOT automatically Data Success

Maximum
→ NOT automatically Best

Cheapest
→ NOT automatically Best

Independent branch
→ NOT automatically blocked

Production
→ Service Principal + Governance + Recovery + Monitoring
```
