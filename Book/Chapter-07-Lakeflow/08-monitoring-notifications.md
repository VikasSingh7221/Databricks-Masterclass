# 08. Monitoring & Notifications

> **Databricks Data Engineer Professional — Certification Notes**
>
> **Source of truth:** Current official Databricks documentation.
>
> **Important:** Monitoring and notification behavior can change. Verify current behavior against official Databricks documentation before the exam.

---

# 1. Why Monitoring Matters

A production Job is not complete merely because it can execute.

You also need to know:

```text
Did it run?
Did it succeed?
How long did it take?
Which task failed?
Why did it fail?
Was it retried?
Did it exceed the SLA?
Did the workload become expensive?
Does someone need to take action?
```

Mental model:

```text
Job Execution
      ↓
Monitoring
      ↓
Detection
      ↓
Notification
      ↓
Investigation
      ↓
Recovery
```

---

# 2. Monitoring vs Notifications

These are different concepts.

## Monitoring

Answers:

> **What is happening / what happened?**

Examples:

```text
Run status
Task status
Duration
Logs
Metrics
Errors
Timeline
```

---

## Notifications

Answers:

> **Who should be informed when something happens?**

Examples:

```text
Job failed
Job succeeded
Job started
Job took too long
Streaming backlog became too high
```

### Mental model

```text
Monitoring
→ Observe

Notification
→ Alert
```

---

# 3. Lakeflow Jobs Monitoring UI

Databricks provides built-in monitoring for Lakeflow Jobs.

You can inspect:

```text
Jobs
Runs
Tasks
Logs
Metrics
Dependencies
Duration
Errors
```

The Jobs UI provides:

- Job history
- Recent runs
- Task-level details
- Run status
- Error information
- Parameters
- Duration
- Timeline
- Graph view
- List view

Official documentation:

https://docs.databricks.com/aws/en/jobs/monitor

---

# 4. Jobs & Pipelines — Runs Tab

The Runs tab provides a view of running and recently completed Jobs and pipelines.

Current Databricks documentation states that the Runs tab can be filtered by:

```text
Job / pipeline name
Type
Run as user
Run ID
Start time
Run status
```

The UI also provides:

```text
Finished runs count
Top error types
Recent runs
```

This is useful for quickly identifying operational problems.

---

# 5. Run History Retention

Current Databricks documentation states that the Jobs UI retains Job run history for:

```text
60 days
```

If historical information is required beyond that period, preserve/export the required information elsewhere.

### Important distinction

```text
UI Job run history
→ 60 days

System.lakeflow tables
→ 365-day free retention
```

These are not the same retention mechanism.

---

# 6. Job Run Statuses

Current Databricks documentation lists Job Run statuses including:

```text
Queued
Pending
Running
Skipped
Succeeded
Succeeded with failures
Failed
Timed Out
Canceling
Canceled
```

These states are important for Professional-level scenario questions.

---

# 7. Succeeded

A Job Run is:

```text
SUCCEEDED
```

when the relevant Job Run success criteria are satisfied.

Current Databricks documentation explains Job Run success using **leaf-task outcomes**.

---

# 8. Succeeded With Failures

This is a major Professional exam concept.

A Job Run can be:

```text
Succeeded with failures
```

when:

```text
Some tasks failed
```

but:

```text
All leaf tasks succeeded
```

### Example

```text
A → B
```

Suppose:

```text
A = FAILED
B = SUCCESS
```

and B is the leaf task.

The Job Run can be:

```text
Succeeded with failures
```

rather than:

```text
Failed
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/monitor

---

# 9. Leaf Tasks

A **leaf task** is:

> A task that has no downstream dependencies.

Example:

```text
A → B → C
```

Then:

```text
C = leaf task
```

because nothing depends on C.

---

# 10. Why Leaf Tasks Matter

Databricks determines Job Run success based on the outcomes of the Job's leaf tasks.

Therefore:

```text
Task failure
≠
automatically Job Run failure
```

You must understand:

```text
Which tasks are leaf tasks?
What are their outcomes?
```

---

# 11. Example: Leaf Task Logic

Consider:

```text
       A
       ↓
       B
```

If:

```text
A = FAILED
B = SUCCESS
```

B is the leaf task.

The Job Run can be:

```text
Succeeded with failures
```

Now consider:

```text
A → B
```

where:

```text
B = FAILED
```

B is a leaf task.

Therefore:

```text
Job Run = FAILED
```

---

# 12. Task-Level Monitoring

The Job Run details page provides information about individual tasks.

You can inspect:

```text
Task name
Task type
Status
Start time
End time
Duration
Dependencies
Logs
Compute
Source code
Metrics
```

This helps isolate the exact location of a failure.

---

# 13. Graph View

The graph view displays the workflow structure.

Example:

```text
Extract
   ↓
Validate
   ↓
Transform
   ↓
Load
```

It helps you understand:

```text
Dependencies
Branches
Task status
Execution path
```

This is useful when troubleshooting complex DAGs.

---

# 14. Timeline View

The Timeline view helps identify:

```text
Long-running tasks
Parallel execution
Task overlap
Dependency delays
Critical path
```

Example:

```text
A ───────────────
B ───────
C ───────────
D                ─────
```

You can visually determine where time is being spent.

---

# 15. Why Timeline Matters

Suppose:

```text
Job duration = 2 hours
```

You discover:

```text
Task A = 10 min
Task B = 15 min
Task C = 90 min
Task D = 5 min
```

The obvious optimization target is:

```text
Task C
```

Monitoring helps you optimize based on evidence rather than guessing.

---

# 16. Task Duration

Monitor:

```text
Current duration
Historical duration
Expected duration
```

A task that normally takes:

```text
10 minutes
```

but suddenly takes:

```text
90 minutes
```

may indicate:

```text
Data growth
Data skew
Infrastructure issue
Source slowdown
Target contention
Code regression
```

---

# 17. Expected Completion Time

Lakeflow Jobs allows you to configure an expected completion time.

If the Job or task exceeds the configured duration:

```text
Duration warning
```

can be generated.

This is useful for:

```text
SLA monitoring
Performance monitoring
Early warning
Operational alerting
```

---

# 18. Duration Warning

Current notification configuration supports:

```text
Duration Warning
```

A notification can be sent when a Job or task exceeds the configured duration threshold.

This is different from:

```text
Task Timeout
```

---

# 19. Duration Warning vs Timeout

Very important distinction.

## Duration Warning

```text
Job is taking longer than expected
```

The workflow can continue.

Example:

```text
Expected = 2 hours
Actual = 2h 10m
```

Notification:

```text
WARNING
```

---

## Timeout

```text
Task/Job exceeds allowed execution limit
```

The run can terminate.

### Mental model

```text
Duration Warning
→ "This is slower than expected."

Timeout
→ "This is not allowed to run this long."
```

---

# 20. Job Notifications

Current Databricks documentation supports notifications for:

```text
Start
Success
Failure
Duration Warning
Streaming Backlog
```

These can be configured at the:

```text
Job level
```

and notifications can also be configured for:

```text
Individual tasks
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/notifications

---

# 21. Start Notification

A notification can be sent when:

```text
Job Run starts
```

Useful when:

```text
A critical workflow begins
```

or when operators need visibility into execution.

However, don't automatically alert on every routine start.

---

# 22. Success Notification

A notification can be sent when:

```text
Job completes successfully
```

This can be useful for:

```text
Critical business pipelines
Data delivery confirmation
Operational workflows
```

But excessive success notifications can create alert noise.

---

# 23. Failure Notification

A notification can be sent when:

```text
Job Run fails
```

This is one of the most important operational notifications.

Typical flow:

```text
Job failure
     ↓
Notification
     ↓
Engineer investigates
     ↓
Fix
     ↓
Repair
```

---

# 24. Important Notification Trap — Retries

Current Databricks documentation states:

> **Job-level failure notifications are not sent when failed tasks are retried.**

If you need notification after each failed task attempt:

```text
Task-level notification
```

should be used.

This is a very important Professional exam detail.

---

# 25. Job-Level vs Task-Level Notifications

## Job-level

Useful for:

```text
Job failed
Job succeeded
Job started
Job exceeded duration
```

---

## Task-level

Useful when you need:

```text
Individual task failure
Individual retry failure
Task-specific monitoring
```

### Mental model

```text
Job notification
→ Job-level event

Task notification
→ Task-level event
```

---

# 26. Succeeded With Failures and Notifications

Current Databricks documentation has an important behavior:

A Job Run with:

```text
Succeeded with failures
```

is considered a:

```text
successful state
```

for notification configuration.

Therefore, if you want notification for:

```text
Succeeded with failures
```

select:

```text
Success
```

notification.

### Exam trap

Do not assume:

```text
Succeeded with failures
→ Failure notification
```

---

# 27. Third-Party Notifications

Databricks supports notification destinations including:

```text
Email
Slack
Microsoft Teams
PagerDuty
HTTP webhooks
```

System destinations must be configured by an administrator where required.

---

# 28. Notification Destinations

A destination can be configured centrally and then used by Jobs.

Conceptually:

```text
Administrator
     ↓
Notification Destination
     ↓
Job
     ↓
Event
     ↓
Notification
```

This separates:

```text
Destination configuration
```

from:

```text
Job-specific notification rules
```

---

# 29. Notification Destination Limits

Current Databricks documentation states that for each Job or task:

```text
Maximum 3 system destinations
```

can be configured for each notification event type.

Example:

```text
Failure
 ├── Slack
 ├── Teams
 └── PagerDuty
```

---

# 30. Email Notifications

Email addresses can be configured directly for Job notifications.

Use email when:

```text
Individual ownership
Small operational team
Simple alerts
```

For larger operational environments, centralized notification destinations may be more appropriate.

---

# 31. Slack / Teams

Slack and Microsoft Teams can receive Job notifications through configured system destinations.

Important:

> Do not build automation that depends on the exact formatting/content of Slack or Teams notification messages.

Databricks notes that their message content may change.

If a strict schema is required:

```text
Use a user-defined webhook
```

---

# 32. Webhooks

Webhooks are useful when an external system needs a controlled notification format.

Conceptually:

```text
Databricks Job
      ↓
Webhook
      ↓
External system
```

Examples:

```text
Incident management
Custom alerting
Internal monitoring platform
Automation service
```

---

# 33. Streaming Backlog Notification

For streaming workloads, notifications can be configured based on:

```text
Streaming backlog
```

A notification can be generated when the backlog metric exceeds a configured threshold.

This is different from:

```text
Job failure
```

because the Job may still be running successfully while falling behind.

---

# 34. Streaming Backlog Behavior

Current documentation states:

```text
Average backlog over 10 minutes
```

is used for the threshold evaluation.

To prevent excessive notifications:

```text
30-minute interval
```

is used before another notification is sent while the backlog remains high.

### Mental model

```text
Job healthy technically
      ↓
Backlog increasing
      ↓
Threshold exceeded
      ↓
Operational warning
```

---

# 35. Failure Notification vs Backlog Notification

### Failure

```text
Job/Task cannot successfully execute
```

### Backlog

```text
Streaming workload is running
but falling behind
```

A streaming Job can therefore be:

```text
RUNNING
+
HIGH BACKLOG
```

without being:

```text
FAILED
```

---

# 36. Monitoring With System Tables

Databricks provides Lakeflow system tables for monitoring Jobs across an account.

Current tables are in:

```text
system.lakeflow
```

Official documentation:

https://docs.databricks.com/aws/en/admin/system-tables/jobs

---

# 37. Current Lakeflow Job System Tables

Current documentation lists:

```text
system.lakeflow.jobs
system.lakeflow.job_tasks
system.lakeflow.job_run_timeline
system.lakeflow.job_task_run_timeline
```

There are additional pipeline-related tables in the same schema.

---

# 38. system.lakeflow.jobs

Tracks:

```text
Jobs created in the account
```

Useful for:

```text
Job inventory
Job metadata
Job-level analysis
```

---

# 39. system.lakeflow.job_tasks

Tracks:

```text
Job tasks
```

Useful for:

```text
Task-level analysis
Task metadata
Task monitoring
```

---

# 40. system.lakeflow.job_run_timeline

Tracks:

```text
Job runs
Run timing
Run metadata
```

This is useful for analyzing:

```text
Job duration
Run frequency
Historical performance
```

---

# 41. system.lakeflow.job_task_run_timeline

Tracks:

```text
Task runs
Task timing
Compute/resource information
```

Useful for:

```text
Slow task identification
Task-level performance analysis
Resource utilization
```

---

# 42. System Table Retention

Current documentation states that the Lakeflow Jobs system tables have:

```text
365 days
```

of free retention.

This differs from:

```text
Jobs UI run history = 60 days
```

### Memorize

```text
UI
→ 60 days

system.lakeflow
→ 365 days
```

---

# 43. System Tables and Cost

System tables can be joined with billing information to analyze:

```text
Job cost
Job performance
Resource utilization
```

Current Databricks documentation provides examples for monitoring Job costs and performance.

Official documentation:

https://docs.databricks.com/aws/en/admin/system-tables/jobs-cost

---

# 44. Why Cost Monitoring Matters

Suppose:

```text
Job runtime improved:
3 hours → 1 hour
```

but:

```text
Concurrency increased dramatically
```

and compute cost doubled.

The performance improvement may not be worth the additional cost.

Therefore monitor:

```text
Runtime
+
Compute usage
+
Cost
```

together.

---

# 45. Operational Dashboard

A production monitoring dashboard might track:

```text
Job success rate
Failure rate
Average duration
P95 duration
SLA breaches
Retry count
Task failures
Top errors
Compute usage
Cost
Streaming backlog
```

This gives a broader view than individual run inspection.

---

# 46. Top Error Types

The Jobs UI provides a view of frequent errors for selected runs.

This can help identify recurring problems:

```text
Permission errors
Timeouts
Out-of-memory errors
Source connection errors
SQL errors
```

Repeated errors suggest:

```text
Systemic problem
```

rather than isolated transient failure.

---

# 47. Monitoring a Failed Run

Recommended workflow:

```text
Job failed
   ↓
Open Runs
   ↓
Identify failed run
   ↓
Open run details
   ↓
Inspect graph
   ↓
Find failed task
   ↓
Inspect task logs
   ↓
Inspect metrics
   ↓
Identify root cause
```

---

# 48. Monitoring With Timeline

If a Job is slow:

```text
Open Timeline
      ↓
Find longest-running tasks
      ↓
Check parallelism
      ↓
Check dependencies
      ↓
Identify bottleneck
```

Don't immediately increase concurrency.

First determine:

```text
Where is time actually being spent?
```

---

# 49. Monitoring + For Each

For Each workloads should monitor:

```text
Number of iterations
Successful iterations
Failed iterations
Retry count
Slow iterations
Overall duration
```

If:

```text
499 iterations = 1 minute
1 iteration = 45 minutes
```

the problem may be:

```text
Data skew
```

rather than insufficient global concurrency.

---

# 50. Monitoring + SLA

Suppose:

```text
SLA = 2 hours
```

Monitor:

```text
Actual duration
Expected duration
SLA breach count
Trend over time
```

Example:

```text
Monday = 1h 20m
Tuesday = 1h 30m
Wednesday = 1h 55m
Thursday = 2h 20m
```

This trend indicates a potential performance regression.

---

# 51. Notification Strategy

A good notification strategy should be:

```text
Actionable
Specific
Reliable
Low-noise
Business-aware
```

Avoid:

```text
Alert on every tiny event
```

because alert fatigue can cause important incidents to be ignored.

---

# 52. Example: Good Failure Notification

```text
Production Customer Gold
Status: FAILED
Task: Load Customer Gold
Run ID: 123456
Error: Permission denied
Action: Repair after permission is restored
```

The notification should help the operator know:

```text
What happened?
Where?
Which run?
What should I investigate?
```

---

# 53. Example: Bad Notification Strategy

Alert every time:

```text
Task retry attempt 1
Task retry attempt 2
Task retry attempt 3
```

This can create:

```text
3 alerts
```

for one transient issue.

A better design may alert after:

```text
Retries exhausted
```

unless task-level failure visibility is specifically required.

---

# 54. Critical vs Non-Critical Jobs

Not every Job should have the same notification policy.

### Critical Job

```text
Failure → immediate alert
Duration breach → alert
```

### Low-priority Job

```text
Failure → dashboard
```

The notification strategy should reflect:

```text
Business impact
Recovery urgency
SLA
```

---

# 55. Notification Routing

A mature organization might route:

```text
Critical production failure
→ PagerDuty

Operational warning
→ Slack / Teams

Routine completion
→ Email

Analytics
→ Dashboard
```

The exact destination depends on organizational requirements.

---

# 56. Monitoring + Repair

Monitoring identifies:

```text
What failed?
```

Repair provides:

```text
How to recover?
```

Architecture:

```text
Execution
   ↓
Monitoring
   ↓
Failure detected
   ↓
Notification
   ↓
Root-cause fix
   ↓
Repair
   ↓
Verify
```

---

# 57. Monitoring + Run Parameters

The Runs UI can display Job Run parameters.

This is especially useful for debugging:

```text
processing_date
environment
load_type
batch_id
```

Example:

```text
Failure
 ↓
Check run parameters
 ↓
processing_date = wrong date
 ↓
Identify configuration issue
```

---

# 58. Monitoring + Trigger

The Runs UI can show how a Job was launched.

Examples include:

```text
Schedule
API request
Manual start
```

This helps answer:

> **Why did this run happen?**

---

# 59. Monitoring + Job Concurrency

If a Job unexpectedly doesn't execute, investigate:

```text
Maximum concurrent runs
Queueing
Existing active runs
Workspace capacity
```

A run may be:

```text
Queued
```

or:

```text
Skipped
```

depending on the configuration and capacity conditions.

---

# 60. Monitoring + Queueing

Suppose:

```text
Max concurrent runs = 1
```

and:

```text
Run 1 = RUNNING
Run 2 = QUEUED
```

Monitoring should make this visible.

The operator should understand:

```text
The Job isn't failing.
It is waiting for capacity.
```

---

# 61. Monitoring + Task States

A task may be:

```text
SUCCESS
FAILED
SKIPPED
DISABLED
```

and other run states can occur depending on the workflow.

Don't interpret every non-success state as:

```text
FAILED
```

Always determine why the task did not execute successfully.

---

# 62. Monitoring + Control Flow

Suppose:

```text
Validation
   │
   ├── PASS → Gold
   │
   └── FAIL → Alert
```

Monitoring should show:

```text
Validation = FAILED
Gold = SKIPPED/EXCLUDED as applicable
Alert = SUCCESS
```

The workflow may therefore be operating exactly as designed.

A skipped downstream task is not necessarily a Job malfunction.

---

# 63. Professional Scenario: Failure Notification

### Requirement

> Notify the team when the Job ultimately fails, but don't generate an alert for every automatic retry.

Good approach:

```text
Job-level Failure notification
```

because current Databricks behavior does not send the Job-level failure notification for each retry attempt.

---

# 64. Professional Scenario: Every Task Failure

### Requirement

> Notify the team after every failed task attempt, including retries.

Use:

```text
Task-level notification
```

because Job-level notifications are not sent for each failed retry attempt.

---

# 65. Professional Scenario: SLA Monitoring

### Requirement

> Alert if a Job runs longer than 2 hours but allow it to continue.

Use:

```text
Expected duration
+
Duration Warning notification
```

Not:

```text
Timeout = 2 hours
```

because timeout would terminate the run rather than simply warn.

---

# 66. Professional Scenario: Streaming Lag

### Requirement

> Alert when a streaming workload falls behind significantly.

Use:

```text
Streaming backlog notification
```

rather than waiting for the Job to fail.

---

# 67. Professional Scenario: Historical Monitoring

### Requirement

> Analyze Job performance over the last year.

Use:

```text
system.lakeflow
```

because the UI's normal Job run history is limited to 60 days, while current Lakeflow Jobs system tables provide 365 days of free retention.

---

# 68. Professional Scenario: Cost Analysis

### Requirement

> Determine whether increasing Job concurrency reduced runtime at an acceptable cost.

Use:

```text
Job monitoring
+
Lakeflow system tables
+
Billing/cost information
```

Analyze:

```text
Runtime
+
Resource usage
+
Cost
```

---

# 69. Monitoring Decision Framework

When a scenario asks how to monitor a Job:

```text
1. Need current run status?
      ↓
   Jobs UI

2. Need task-level debugging?
      ↓
   Run details / logs / graph

3. Need performance analysis?
      ↓
   Timeline / metrics

4. Need alerting?
      ↓
   Notifications

5. Need every failed retry attempt?
      ↓
   Task notification

6. Need Job-level failure?
      ↓
   Job notification

7. Need SLA warning?
      ↓
   Duration Warning

8. Need streaming lag alert?
      ↓
   Streaming Backlog

9. Need long-term analytics?
      ↓
   system.lakeflow

10. Need cost analysis?
      ↓
   system.lakeflow + billing
```

---

# 70. Exam Decision Rules

### Rule 1

> **Monitoring tells you what happened; notifications tell you when to alert someone.**

### Rule 2

> **Use the Jobs UI for interactive run/task investigation.**

### Rule 3

> **Use Timeline view to identify long-running tasks and overlap.**

### Rule 4

> **Job-level failure notifications are not sent for each failed retry attempt.**

### Rule 5

> **Use task-level notifications when individual task failures/retries must generate notifications.**

### Rule 6

> **Succeeded with failures is considered successful for notification selection, so select Success to receive that notification.**

### Rule 7

> **Use Duration Warning when you want an alert without necessarily terminating the Job.**

### Rule 8

> **Use Timeout when the workload should be terminated after exceeding the configured limit.**

### Rule 9

> **Use Streaming Backlog notification to detect lag even when the workload has not failed.**

### Rule 10

> **Current Jobs UI run history is 60 days.**

### Rule 11

> **Current Lakeflow system tables provide 365 days of free retention.**

### Rule 12

> **Use system tables for account-level historical Job analysis and cost/performance monitoring.**

---

# 71. Common Exam Traps

## Trap 1

> Job failure notification is sent for every retry.

❌ Incorrect.

Use task notifications if every failed task attempt must generate an alert.

---

## Trap 2

> Succeeded with failures triggers Failure notification.

❌ Incorrect.

For notification purposes, it is considered successful.

---

## Trap 3

> Duration warning stops the Job.

❌ Incorrect.

It warns when the configured duration threshold is exceeded.

---

## Trap 4

> Timeout and duration warning are the same.

❌ Incorrect.

```text
Duration Warning
→ Alert

Timeout
→ Termination
```

---

## Trap 5

> Job UI retains unlimited Job history.

❌ Incorrect.

Current documented UI retention is:

```text
60 days
```

---

## Trap 6

> system.lakeflow contains only current Job information.

❌ Incorrect.

It provides historical Job/task/run information with current documented free retention of:

```text
365 days
```

---

## Trap 7

> A failed task always means the entire Job failed.

❌ Incorrect.

Leaf-task outcomes determine Job Run success, and `Succeeded with failures` is possible.

---

## Trap 8

> Streaming backlog means the Job has failed.

❌ Incorrect.

The Job can still be running while backlog exceeds the configured threshold.

---

# 72. Quick Reference

| Requirement | Mechanism |
|---|---|
| View current Job status | Jobs UI |
| Investigate failed task | Run details + logs |
| Understand dependencies | Graph view |
| Find slow tasks | Timeline view |
| Alert on Job start | Start notification |
| Alert on Job success | Success notification |
| Alert on Job failure | Failure notification |
| Alert on every failed retry | Task notification |
| Alert on slow Job | Duration Warning |
| Terminate long-running task | Timeout |
| Alert on streaming lag | Streaming Backlog |
| Historical Job analytics | `system.lakeflow` |
| Cost/performance analysis | System tables + billing |
| Account-wide monitoring | Lakeflow system tables |

---

# 73. Final Mental Model

```text
                    JOB
                     │
                  RUNNING
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      MONITOR                NOTIFY
          │                     │
          ↓                     ↓
    ┌────────────┐       ┌─────────────┐
    │ UI         │       │ Start       │
    │ Graph      │       │ Success     │
    │ Timeline   │       │ Failure     │
    │ Logs       │       │ Duration    │
    │ Metrics    │       │ Backlog     │
    └────────────┘       └─────────────┘
          │
          ↓
    system.lakeflow
          │
          ↓
 Historical Analysis
          │
          ↓
 Performance / Cost
```

---

# 74. One-Line Rules to Memorize

```text
Monitoring
→ Observe execution.

Notifications
→ Alert people/systems.

Jobs UI
→ Interactive investigation.

Graph
→ Dependency structure.

Timeline
→ Duration + overlap + bottlenecks.

Job notification
→ Job-level event.

Task notification
→ Task-level event.

Duration Warning
→ Alert on slow execution.

Timeout
→ Stop execution after limit.

Streaming Backlog
→ Detect streaming lag.

Succeeded with failures
→ Some tasks failed, leaf tasks succeeded.

UI history
→ 60 days.

system.lakeflow
→ 365 days free retention.

System tables
→ Historical + account-level analysis.

Cost monitoring
→ Runtime + resources + billing.
```

---

# 75. Official Documentation

Use these as the primary references for this chapter:

- [Monitor Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/monitor)
- [Add notifications on a Job](https://docs.databricks.com/aws/en/jobs/notifications)
- [Jobs system table reference](https://docs.databricks.com/aws/en/admin/system-tables/jobs)
- [Monitor Job costs and performance with system tables](https://docs.databricks.com/aws/en/admin/system-tables/jobs-cost)
- [Lakeflow Jobs overview](https://docs.databricks.com/aws/en/jobs/)

---

## Chapter Status

**08. Monitoring & Notifications — COMPLETE ✅**

### Key verified facts

```text
Job notifications:
→ Start
→ Success
→ Failure
→ Duration Warning
→ Streaming Backlog

Job-level failure notification:
→ Not sent for each retry attempt

Task-level notification:
→ Use when individual task failures/retries must alert

Succeeded with failures:
→ Considered successful for notification selection

Jobs UI history:
→ 60 days

Lakeflow system tables:
→ system.lakeflow
→ 365 days free retention

System tables:
→ Historical/account-level analysis

Timeline:
→ Identify long-running tasks and overlap

Duration Warning:
→ Alert

Timeout:
→ Termination

Streaming Backlog:
→ Detect lag without requiring Job failure
```

