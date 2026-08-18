# 06. Triggers & Scheduling

> **Databricks Data Engineer Professional — Certification Notes**
>
> **Source of truth:** Current official Databricks documentation.
>
> **Important:** Product behavior can change. For exam preparation, verify trigger-specific behavior against the current Databricks documentation.

---

# 1. What Is a Job Trigger?

A **trigger** determines when a Lakeflow Job Run should start.

Mental model:

```text
Trigger
   ↓
Starts Job Run
   ↓
Tasks execute according to dependencies/control flow
```

A trigger answers:

> **WHEN should the Job start?**

It does **not** determine the order in which tasks execute.

Official documentation:

https://docs.databricks.com/aws/en/jobs/triggers

---

# 2. Trigger vs Dependency

This distinction is critical.

## Trigger

```text
When should the Job Run start?
```

Example:

```text
Every day at 2 AM
```

---

## Dependency

```text
When can Task B execute relative to Task A?
```

Example:

```text
A → B
```

### Mental model

```text
TRIGGER
→ Starts the Job

DEPENDENCY
→ Controls task relationships

RUN IF
→ Controls whether a task runs based on upstream outcomes
```

---

# 3. Current Lakeflow Jobs Trigger Types

Current Databricks documentation lists these automatic trigger types:

1. Scheduled
2. Table update
3. File arrival
4. Model update
5. Continuous

Jobs can also be run:

6. Manually
7. Programmatically / through external orchestration

Official documentation:

https://docs.databricks.com/aws/en/jobs/triggers

---

# 4. Trigger Type Overview

| Trigger | Starts Job when... |
|---|---|
| Scheduled | A time-based schedule occurs |
| Table update | Configured source table(s) are updated |
| File arrival | New files arrive in a monitored location |
| Model update | A configured Unity Catalog model event occurs |
| Continuous | A new Job Run is started after the previous run completes/fails |
| Manual | User explicitly starts the Job |
| Programmatic | External system/API starts the Job |

---

# 5. Scheduled Trigger

A scheduled trigger starts a Job according to a time-based schedule.

Example:

```text
Every day at 2 AM
```

Architecture:

```text
2 AM
 ↓
Job Run
 ↓
Tasks
```

Use scheduled triggers when the requirement is primarily:

> **Run at a defined time or recurring interval.**

---

# 6. Scheduled Trigger Examples

### Daily

```text
Every day at 2 AM
```

### Hourly

```text
Every hour
```

### Periodic interval

```text
Every N minutes/hours
```

The exact schedule configuration can be represented through the Jobs UI or API.

---

# 7. Scheduled Trigger — When to Use

Good use cases:

```text
Daily ETL
Nightly warehouse load
Daily reporting
Periodic aggregation
Regular data-quality checks
```

Example:

```text
Source
  ↓
2 AM Scheduled Trigger
  ↓
Ingestion
  ↓
Validation
  ↓
Transformation
```

---

# 8. Scheduled Trigger Is Not Event-Driven

Suppose files arrive randomly:

```text
10:05
10:47
11:03
13:22
```

A schedule such as:

```text
Every hour
```

does not directly express:

> Run when a file arrives.

For that requirement, consider:

```text
File Arrival Trigger
```

---

# 9. File Arrival Trigger

A **file arrival trigger** starts a Job when new files arrive in a monitored Unity Catalog storage location.

Current Databricks documentation supports monitoring:

- Unity Catalog external locations
- Unity Catalog volumes
- Root paths
- Subpaths

Official documentation:

https://docs.databricks.com/aws/en/jobs/file-arrival-triggers

---

# 10. Why Use File Arrival Trigger?

Consider:

```text
Files arrive irregularly
```

A scheduled Job might repeatedly start even when no new data is available.

Example:

```text
Schedule:
Every 15 minutes

Actual file arrivals:
09:02
11:47
15:23
```

The scheduled workflow may perform unnecessary checks/runs.

A file arrival trigger is better aligned with:

> **Start when new files arrive.**

---

# 11. File Arrival Trigger Behavior

Databricks documents that file arrival triggers make a **best-effort check approximately every minute**, although actual timing can be affected by underlying cloud storage performance.

Important:

> It is not an exact real-time guarantee.

Think:

```text
File arrives
   ↓
Trigger detects arrival
   ↓
Job Run starts
```

but with detection/checking latency.

---

# 12. File Arrival Trigger — Important Configuration

Two important advanced options are available:

```text
Minimum time between triggers
```

and:

```text
Wait after last change
```

These solve different problems.

---

# 13. Minimum Time Between Triggers

This controls the minimum interval between Job Runs created by the file-arrival trigger.

Think:

> **Cooldown**

Example:

```text
Minimum time between triggers = 900 seconds
```

means the trigger limits run creation to at most approximately one run per 15 minutes.

If more files arrive during the cooldown:

```text
Files arrive
     ↓
Cooldown
     ↓
No immediate additional run
```

---

# 14. Why Use Minimum Time Between Triggers?

Suppose files arrive continuously:

```text
10:00 file
10:01 file
10:02 file
10:03 file
10:04 file
...
```

Without rate control, the workflow could be triggered frequently.

A cooldown can reduce excessive Job Run creation.

Mental model:

```text
Many arrivals
     ↓
Cooldown
     ↓
Controlled Job Run frequency
```

---

# 15. Wait After Last Change

This setting is used for **debouncing**.

It waits for a period after the most recent file arrival before starting the Job Run.

Important behavior:

> Every new file arrival resets the timer.

Example:

```text
File 1 arrives
    ↓
Timer starts

File 2 arrives
    ↓
Timer resets

File 3 arrives
    ↓
Timer resets

No more files
    ↓
Timer expires
    ↓
Job starts
```

---

# 16. Why Use Wait After Last Change?

Suppose files arrive as a batch:

```text
10:00 file1
10:00 file2
10:01 file3
10:01 file4
10:02 file5
```

Requirement:

> Process the entire batch in one Job Run.

Configure a suitable:

```text
Wait after last change
```

so the Job waits until the batch has finished arriving.

---

# 17. Cooldown vs Debouncing

This is worth memorizing.

| Setting | Mental model | Purpose |
|---|---|---|
| Minimum time between triggers | Cooldown | Limit frequency of runs |
| Wait after last change | Debounce | Wait until arrivals settle |

### Easy memory

```text
Minimum time
→ "Don't run too often."

Wait after last change
→ "Wait until files stop arriving."
```

---

# 18. Combining Both File Arrival Options

You can configure both.

Example:

```text
Minimum time between triggers = 900 sec
Wait after last change = 60 sec
```

Behavior:

```text
Files arrive
     ↓
Wait until 60 sec after last arrival
     ↓
Start Job
     ↓
Cooldown prevents another run
     ↓
Next eligible run after cooldown
```

This can help when you want:

- Complete batches
- Controlled run frequency

---

# 19. Table Update Trigger

A **table update trigger** starts a Job when configured source tables are updated.

Official documentation:

https://docs.databricks.com/aws/en/jobs/trigger-table-update

Use this when the requirement is:

> **Run when the source table has been updated.**

---

# 20. Table Update Trigger

A trigger can monitor one or more tables.

It can be configured so that the Job runs when:

```text
One monitored table updates
```

or:

```text
All monitored tables update
```

depending on the trigger configuration.

---

# 21. What Counts as a Table Update?

Current documentation states that table update triggers can respond to data changes such as:

```text
Updates
Merges
Deletes
```

This makes the trigger useful when the event of interest is a table change rather than a file arrival.

---

# 22. Table Update vs File Arrival

This distinction is important.

### File Arrival

Requirement:

> "Run when new files arrive."

Use:

```text
File Arrival Trigger
```

### Table Update

Requirement:

> "Run when the source table is updated."

Use:

```text
Table Update Trigger
```

---

# 23. Model Update Trigger

Lakeflow Jobs can also trigger based on Unity Catalog model events.

Current documentation lists events such as:

```text
Model created
Model version ready
Model alias set
```

The model update trigger is currently documented as **Beta**.

Use it when the workflow depends on model lifecycle changes.

---

# 24. Continuous Trigger

A **continuous trigger** is used to keep a Job running continuously.

Current Databricks documentation recommends continuous mode for:

> **Always-on streaming workloads.**

Official documentation:

https://docs.databricks.com/aws/en/jobs/continuous

Mental model:

```text
Job Run
   ↓
Completes / fails
   ↓
Another Job Run
   ↓
Completes / fails
   ↓
Another Job Run
   ↓
...
```

---

# 25. Continuous vs Scheduled

### Scheduled

```text
2 AM
 ↓
Run
```

Then wait until the next scheduled time.

### Continuous

```text
Run
 ↓
Complete/fail
 ↓
New run
 ↓
Complete/fail
 ↓
New run
 ↓
...
```

### Use continuous when:

```text
Always-on processing
Continuous streaming workload
```

---

# 26. Important Continuous Job Limitation

Current Databricks documentation states:

> There can be only **one running instance** of a continuous Job.

This is different from a normal Job where maximum concurrent runs can be configured.

---

# 27. Continuous Jobs and Task Dependencies

This is a major current behavior to remember.

Current documentation states:

> **You cannot use task dependencies with a continuous Job.**

Therefore, do not design:

```text
A → B → C
```

as a continuous Job if the current continuous-job restrictions apply.

This is an important exam/detail check.

---

# 28. Continuous Jobs and Retry Behavior

Continuous Jobs have specialized failure handling.

Current documentation states that continuous Jobs automatically retry the entire Job on failure using an **exponential backoff** algorithm.

This differs from normal task retry configuration.

Conceptually:

```text
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

---

# 29. Continuous Jobs and Task Retry Mode

Current documentation allows a task retry mode for continuous Jobs:

```text
On failure
```

or:

```text
Never
```

The default task retry mode for continuous mode is documented as:

```text
On failure
```

However, continuous-job-level failure handling still uses exponential backoff.

---

# 30. Continuous Jobs and Serverless

Current documentation notes that continuous scheduling on serverless compute works with **bounded Structured Streaming triggers**, such as:

```text
Trigger.AvailableNow
```

Time-based Structured Streaming triggers such as:

```text
Trigger.ProcessingTime
Trigger.Continuous
```

are not supported on serverless compute.

### Exam principle

Do not assume:

```text
Continuous Job
=
any Structured Streaming trigger
=
serverless
```

Check the specific supported configuration.

---

# 31. Manual Trigger

A Job can be run manually using:

```text
Run now
```

This is useful for:

- Testing
- Operational reruns
- Ad-hoc execution
- Backfills
- Troubleshooting

Manual execution does not require a recurring trigger.

---

# 32. Programmatic Trigger

Jobs can also be triggered programmatically through:

- REST API
- Databricks CLI
- External orchestration tools
- Other supported integrations

Mental model:

```text
External system
      ↓
API / orchestration
      ↓
Databricks Job
      ↓
Job Run
```

---

# 33. Trigger vs External Orchestrator

Sometimes Databricks is not responsible for the entire workflow.

Example:

```text
Airflow
   ↓
Databricks Job
```

The external orchestrator can trigger the Job.

The Job still controls its own internal task execution.

Therefore:

```text
External orchestrator
→ starts Job

Lakeflow Jobs
→ orchestrates tasks inside Job
```

---

# 34. Multiple Triggers

A Job can have more than one trigger.

For example:

```text
Scheduled Trigger
       │
       ├──→ Job
       │
File Arrival Trigger
       │
       └──→ Job
```

Current Databricks system tables can represent a Job with multiple triggers using a trigger type of:

```text
MULTIPLE
```

in relevant system-table records.

---

# 35. Pausing a Trigger

A configured trigger can be paused.

When a trigger is paused:

```text
Current active run
→ continues

New trigger-based runs
→ do not start
```

When resumed:

```text
Trigger
→ resumes its configured behavior
```

This is useful during:

- Maintenance
- Incident response
- Deployment
- Source-system outages
- Planned downtime

---

# 36. Trigger and Active Runs

Suppose:

```text
Job max concurrent runs = 1
```

and:

```text
Run #1 is still active
```

A scheduled trigger fires:

```text
2 AM
 ↓
New run requested
```

But the Job may not be able to start another run because of its concurrency configuration.

Current Databricks documentation states that runs can be skipped when the configured maximum concurrent runs is reached, and queueing can be enabled to queue runs instead.

---

# 37. Trigger + Job Concurrency

This is a major Professional scenario.

Suppose:

```text
Schedule = every 15 minutes
Runtime = 45 minutes
Max concurrent runs = 1
```

Then:

```text
12:00 → Run 1 starts
12:15 → Run 2 requested
12:30 → Run 3 requested
```

If the first run is still active and concurrency is 1:

```text
New runs cannot execute concurrently
```

Depending on queueing configuration, runs may be:

```text
Skipped
```

or:

```text
Queued
```

---

# 38. Queueing

Current Databricks documentation states that queueing can prevent runs from being skipped because of concurrency/resource limits.

For applicable jobs:

```text
Run requested
     ↓
Capacity unavailable
     ↓
Queue
     ↓
Capacity becomes available
     ↓
Run starts
```

Current documentation states that queued runs can remain queued for up to **48 hours** if the relevant conditions persist.

---

# 39. Trigger + Queueing Decision

Suppose:

```text
Job runs every 10 minutes
Runtime = 30 minutes
```

If overlapping runs are not safe:

```text
Max concurrent runs = 1
```

Then decide:

```text
Should missed executions be skipped?
```

or:

```text
Should they wait in a queue?
```

This is an operational/business requirement.

---

# 40. Event Trigger vs Schedule

### Requirement:

> Run every day at 2 AM.

Use:

```text
Scheduled Trigger
```

### Requirement:

> Run when files arrive.

Use:

```text
File Arrival Trigger
```

### Requirement:

> Run when source table changes.

Use:

```text
Table Update Trigger
```

### Requirement:

> Keep processing continuously.

Consider:

```text
Continuous Trigger
```

---

# 41. File Arrival vs Table Update

Scenario:

```text
Raw files
   ↓
Delta table
```

If the requirement is:

> Process as soon as files arrive.

Use:

```text
File Arrival
```

If the requirement is:

> Process when the source table is updated.

Use:

```text
Table Update
```

The correct trigger depends on **what event represents readiness for the downstream workflow**.

---

# 42. Scheduled Trigger vs Continuous

### Scheduled

Good for:

```text
Daily batch
Hourly batch
Nightly ETL
Periodic reports
```

### Continuous

Good for:

```text
Always-on streaming workloads
```

Don't choose continuous simply because:

> "I want the data to be processed frequently."

If periodic batch processing is sufficient:

```text
Scheduled
```

may be more appropriate.

---

# 43. File Arrival vs Continuous Streaming

If files arrive irregularly:

```text
File Arrival Trigger
```

can start processing only when new files arrive.

A continuously running workload may consume resources even when no new data is available, depending on the architecture.

### Professional principle

> **Choose event-driven execution when the business requirement is event-driven.**

---

# 44. Debouncing Batch File Arrival

Scenario:

```text
100 files
```

arrive over:

```text
2 minutes
```

Requirement:

> Process all 100 files together.

Use:

```text
Wait after last change
```

Example:

```text
Wait after last change = 60 sec
```

Conceptually:

```text
File 1 → timer
File 2 → reset
File 3 → reset
...
File 100 → reset

No new files
 ↓
60 sec
 ↓
Job starts
```

---

# 45. Limiting File Trigger Frequency

Scenario:

```text
Files arrive continuously
```

Requirement:

> Never start more than one Job Run every 15 minutes.

Use:

```text
Minimum time between triggers = 900 sec
```

This is a cooldown/rate-limiting behavior.

---

# 46. Combining File Trigger Controls

Requirement:

> Wait for a batch to finish arriving, but also don't trigger more than once every 15 minutes.

Use:

```text
Wait after last change = 60 sec
Minimum time between triggers = 900 sec
```

This combines:

```text
Debouncing
+
Cooldown
```

---

# 47. Trigger Selection Framework

When solving an exam scenario:

```text
What event should start the workflow?
              ↓
       ┌──────┼────────┐
       ↓      ↓        ↓
      Time   Data     Continuous
       ↓      ↓        ↓
   Scheduled  │       Continuous
              │
        ┌─────┴─────┐
        ↓           ↓
      Files       Table
        ↓           ↓
 File Arrival   Table Update
```

---

# 48. Professional Scenario

### Requirement

A source system drops files unpredictably.

Files arrive in batches.

The Job should:

- Start after a batch finishes arriving
- Avoid excessive run frequency
- Process each batch
- Not overlap unsafely

### Good architecture

```text
File Arrival Trigger
        ↓
Wait after last change
        ↓
Minimum time between triggers
        ↓
Job
        ↓
Controlled Job concurrency
```

This combines trigger configuration with Job concurrency.

---

# 49. Another Professional Scenario

### Requirement

A pipeline must process every night at 2 AM.

### Answer

```text
Scheduled Trigger
```

Not:

```text
File Arrival
```

unless the actual requirement is based on file arrival.

---

# 50. Another Professional Scenario

### Requirement

A Job should run whenever a source table is updated.

### Answer

```text
Table Update Trigger
```

Potentially configured to react to:

```text
Any monitored table update
```

or:

```text
All monitored tables updated
```

depending on the requirement.

---

# 51. Another Professional Scenario

### Requirement

An always-on streaming workload should continuously process data.

### Answer

Consider:

```text
Continuous Job
```

But verify:

```text
Streaming trigger compatibility
Compute type
Retry behavior
Task dependency limitations
```

---

# 52. Trigger + Dependency

Suppose:

```text
2 AM
 ↓
Job
 ↓
A → B → C
```

The trigger starts the Job at 2 AM.

Dependencies still determine:

```text
A → B → C
```

### Important

Changing:

```text
2 AM → 3 AM
```

does not change:

```text
A → B → C
```

And changing:

```text
A → B
```

does not change when the Job starts.

---

# 53. Trigger + Run If

Example:

```text
Trigger
   ↓
Job
   ↓
A
   ↓
B
```

The trigger determines:

```text
When Job starts
```

Run If determines:

```text
Whether B runs based on A's outcome
```

Therefore:

```text
Trigger ≠ Run If
```

---

# 54. Trigger + For Each

Example:

```text
File Arrival
     ↓
Job
     ↓
Discover Files
     ↓
For Each
     ↓
Process Files
```

The trigger controls:

```text
When Job starts
```

For Each controls:

```text
How repeated inputs are processed
```

---

# 55. Trigger + Incremental Processing

Event-driven execution does not automatically guarantee incremental processing.

Example:

```text
File Arrival
     ↓
Job
     ↓
Full table scan
```

The Job is event-triggered but may still process all historical data.

A better design might be:

```text
File Arrival
     ↓
Identify new files
     ↓
Incremental processing
```

### Professional principle

> **Trigger mechanism and data-processing strategy are separate design decisions.**

---

# 56. Trigger + Idempotency

Suppose:

```text
File arrival
     ↓
Job starts
```

The workflow should still consider:

```text
Duplicate event
Overlapping runs
Retry
Partial failure
```

Therefore:

```text
Event-driven
      +
Idempotent processing
```

is important for robust production design.

---

# 57. Trigger + Backfill

Suppose the normal Job is:

```text
Scheduled daily
```

but historical data must be reprocessed.

Don't necessarily create a special permanent trigger.

Instead:

```text
Manual/API Run
+
processing_date parameter
```

can support targeted backfill where the workflow is designed for it.

---

# 58. Trigger + Maximum Concurrent Runs

Always ask:

```text
Can two Job Runs safely overlap?
```

If:

```text
YES
```

controlled concurrency may be appropriate.

If:

```text
NO
```

consider:

```text
Max concurrent runs = 1
```

and decide whether queueing is appropriate.

---

# 59. Trigger Design Checklist

Before choosing a trigger, ask:

```text
1. What event represents data readiness?

2. Is the requirement time-based?

3. Is the requirement file-based?

4. Is the requirement table-update-based?

5. Is the workload continuously running?

6. Can runs overlap safely?

7. What happens if a trigger fires while a run is active?

8. Should another run be skipped or queued?

9. Should multiple file arrivals create one run or many?

10. Is batch completion important?

11. Is low latency important?

12. Does the compute/workload support the selected trigger?
```

---

# 60. Exam Decision Rules

### Rule 1

> **Trigger starts the Job; dependencies control task execution order.**

### Rule 2

> **Use Scheduled Trigger for time-based execution.**

### Rule 3

> **Use File Arrival Trigger when new files arriving is the event that should start processing.**

### Rule 4

> **Use Table Update Trigger when source table changes represent readiness.**

### Rule 5

> **Use Continuous mode for appropriate always-on workloads, especially streaming workloads.**

### Rule 6

> **File Arrival `Wait after last change` is a debouncing mechanism.**

### Rule 7

> **File Arrival `Minimum time between triggers` is a cooldown/frequency control.**

### Rule 8

> **Trigger configuration does not replace Job concurrency configuration.**

### Rule 9

> **If overlapping runs are unsafe, explicitly consider maximum concurrent runs and queueing.**

### Rule 10

> **Event-triggered execution does not automatically mean incremental processing.**

---

# 61. Common Exam Traps

## Trap 1

> "A dependency controls when the Job starts."

❌ Wrong.

```text
Trigger → Job start
Dependency → task relationship
```

---

## Trap 2

> "File arrival means the Job starts instantly."

❌ Too strong.

Current documentation describes file-arrival detection as a **best-effort check approximately every minute**, subject to storage performance.

---

## Trap 3

> "Wait after last change limits run frequency."

❌ Not primarily.

```text
Wait after last change
→ debounce

Minimum time between triggers
→ cooldown/frequency limit
```

---

## Trap 4

> "Continuous means unlimited concurrent Job Runs."

❌ Wrong.

Current documentation states a continuous Job can have only **one running instance**.

---

## Trap 5

> "Continuous Job supports normal task dependencies."

❌ Current documentation says continuous Jobs cannot use task dependencies.

---

## Trap 6

> "Scheduled trigger is always better than file arrival."

❌ Wrong.

Choose based on the event that represents data readiness.

---

## Trap 7

> "If a file arrives, the pipeline must process only that file."

❌ Not necessarily.

The trigger determines **when the Job starts**. The Job's processing logic determines **what data it processes**.

---

## Trap 8

> "Increasing maximum concurrent runs fixes trigger delays."

❌ Not automatically.

First determine:

```text
Why is the run waiting?
```

Possible causes include:

```text
Concurrency
Compute capacity
Queueing
Task duration
Source/target bottleneck
```

---

# 62. Quick Reference

| Requirement | Recommended trigger |
|---|---|
| Daily at 2 AM | Scheduled |
| Every hour | Scheduled |
| New files arrive | File Arrival |
| Files arrive in batches | File Arrival + Wait after last change |
| Limit file-trigger frequency | File Arrival + Minimum time between triggers |
| Source table updated | Table Update |
| Model event | Model Update |
| Always-on workload | Continuous |
| Manual execution | Run now |
| External orchestrator | API/programmatic trigger |

---

# 63. Trigger Architecture

```text
                 TRIGGER
                    │
       ┌────────────┼─────────────┐
       ↓            ↓             ↓
     TIME         EVENT        CONTINUOUS
       │            │             │
       ↓       ┌────┴────┐        ↓
  Scheduled    ↓         ↓    Continuous
             Files      Table
               ↓          ↓
          File Arrival  Table Update
               │          │
               └────┬─────┘
                    ↓
                  JOB RUN
                    ↓
             Task Dependencies
                    ↓
                Run If / Flow
                    ↓
                 Tasks
```

---

# 64. Final Mental Model

Memorize:

```text
TRIGGER
→ WHEN should the Job start?

SCHEDULED
→ Time

FILE ARRIVAL
→ New files

TABLE UPDATE
→ Table changes

MODEL UPDATE
→ Model lifecycle event

CONTINUOUS
→ Keep workload running

MANUAL/API
→ Explicit external start
```

Then separately:

```text
DEPENDENCY
→ Which tasks depend on which?

RUN IF
→ Should downstream task execute?

FOR EACH
→ Repeat processing for multiple inputs?

CONCURRENCY
→ How many runs/iterations can execute simultaneously?
```

---

# 65. Professional Scenario Pattern

For a scenario question, use:

```text
Requirement
    ↓
Identify trigger event
    ↓
Choose trigger type
    ↓
Determine run frequency/latency
    ↓
Check overlapping-run safety
    ↓
Configure Job concurrency
    ↓
Consider queueing
    ↓
Design task dependencies
    ↓
Design processing strategy
    ↓
Validate retry/idempotency
```

Do not stop at:

> "Use a file trigger."

Professional questions often test the **second-order consequences**:

```text
File arrival
   ↓
Many arrivals
   ↓
Batching/debounce?
   ↓
Run frequency?
   ↓
Concurrent runs?
   ↓
Queueing?
   ↓
Idempotency?
```

---

# 66. Official Documentation

Use these as the primary references for this chapter:

- [Automate jobs with schedules and triggers](https://docs.databricks.com/aws/en/jobs/triggers)
- [Trigger jobs when new files arrive](https://docs.databricks.com/aws/en/jobs/file-arrival-triggers)
- [Trigger jobs when source tables are updated](https://docs.databricks.com/aws/en/jobs/trigger-table-update)
- [Run jobs continuously](https://docs.databricks.com/aws/en/jobs/continuous)
- [Configure and edit Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/configure-job)
- [Lakeflow Jobs overview](https://docs.databricks.com/aws/en/jobs/)

---

## Chapter Status

**06. Triggers & Scheduling — COMPLETE ✅**

### Key verified facts

```text
Scheduled
→ Time-based

File Arrival
→ New files in monitored UC storage

Table Update
→ Source table changes

Model Update
→ Unity Catalog model events

Continuous
→ Continuously restart runs
→ One running instance
→ No task dependencies
→ Exponential backoff behavior

File Arrival:
Wait after last change
→ Debounce

Minimum time between triggers
→ Cooldown/frequency control

Job concurrency
→ Separate from trigger configuration
```
