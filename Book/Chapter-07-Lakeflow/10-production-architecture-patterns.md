# 10. Production Architecture Patterns

> **Databricks Data Engineer Professional — Certification Notes**
>
> **Source of truth:** Current official Databricks documentation.
>
> This chapter focuses on how to combine Lakeflow Jobs features into reliable production architectures.
>
> **Important distinction:**
>
> ```text
> Official Databricks behavior
> ≠
> Architecture recommendation
> ```
>
> Where a recommendation is an architectural best practice rather than a hard product rule, it is explicitly identified.

---

# 1. What Makes a Job Production-Ready?

A production workflow should not only "work."

It should be:

```text
Reliable
Recoverable
Observable
Secure
Maintainable
Scalable
Cost-conscious
Idempotent
```

A useful mental model:

```text
                 PRODUCTION JOB
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
   Correctness      Reliability      Security
       │               │                │
       ↓               ↓                ↓
  Validation       Retry/Repair      Identity
  Idempotency      Recovery          Permissions
       │               │                │
       └───────────────┼────────────────┘
                       ↓
                 Observability
                       ↓
                 Performance
                       ↓
                     Cost
```

---

# 2. Databricks Production Recommendation

Current Databricks production guidance recommends:

```text
Serverless compute for supported Jobs
Lakeflow Jobs for Databricks orchestration
Service principals for production execution
Unity Catalog-compatible compute
Jobs compute instead of all-purpose compute for classic Jobs
```

These recommendations are designed to improve:

```text
Cost
Security
Reliability
Maintainability
Governance
```

Official production scheduling guidance:

https://docs.databricks.com/aws/en/cheat-sheet/jobs

---

# 3. Production Architecture Principle #1

> **Use Lakeflow Jobs for orchestration when the workload is primarily within Databricks.**

Example:

```text
Source
  ↓
Ingestion
  ↓
Validation
  ↓
Transformation
  ↓
Gold
  ↓
Audit
```

This can be represented as a Lakeflow Job DAG.

Current Databricks guidance recommends using Lakeflow Jobs for orchestration whenever possible for Databricks workloads. :contentReference[oaicite:1]{index=1}

---

# 4. Why Use Native Orchestration?

Instead of unnecessarily building:

```text
External orchestrator
      ↓
Databricks
      ↓
Databricks
      ↓
Databricks
```

you can often use:

```text
Lakeflow Jobs
      ↓
Task A
      ↓
Task B
      ↓
Task C
```

Benefits:

```text
Native dependencies
Native retries
Native parameters
Native monitoring
Native notifications
Native repair
```

---

# 5. When External Orchestration Still Makes Sense

Lakeflow Jobs does not mean:

> "Never use an external orchestrator."

External orchestration can be appropriate when:

```text
Workflow spans multiple platforms
Enterprise orchestration standard requires it
Dependencies exist outside Databricks
Cross-cloud systems must be coordinated
```

Example:

```text
Airflow
   │
   ├── AWS process
   ├── Databricks Job
   └── External API
```

The key question is:

> **Where should orchestration responsibility live?**

---

# 6. Production Identity

Do not run production workflows using a developer's personal identity when avoidable.

Current Databricks guidance recommends:

> **Run production Jobs using a service principal.**

Why?

Suppose:

```text
Job
 ↓
Run as Vikas
```

Then:

```text
Vikas leaves organization
      ↓
Permissions may change
      ↓
Job can stop working
```

Instead:

```text
Job
 ↓
Service Principal
 ↓
Stable production identity
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/privileges

---

# 7. Service Principal Benefits

A service principal provides:

```text
Stable identity
Controlled permissions
Better security
Reduced dependency on individuals
Easier governance
```

It also supports the principle:

> **Give the workload only the permissions it needs.**

---

# 8. Least Privilege

Production Job identity should have only required permissions.

Example:

```text
Customer Gold Job
```

might need:

```text
READ:
bronze.customer

WRITE:
silver.customer
gold.customer
```

It should not automatically receive:

```text
WRITE:
every production table
```

### Professional rule

> **Grant the minimum permissions required for the Job.**

---

# 9. Unity Catalog Governance

Production Jobs should use Unity Catalog-compatible compute configurations.

Current Databricks documentation states:

```text
Serverless compute
→ Unity Catalog compatible

SQL warehouses
→ Unity Catalog compatible

Classic compute
→ Use supported access modes
```

For supported classic workloads, Databricks recommends:

```text
Standard access mode
```

Official documentation:

https://docs.databricks.com/aws/en/jobs/privileges

---

# 10. Production Data Location

Production data should not be casually stored in insecure or poorly governed locations.

Current Databricks production guidance recommends:

> **Do not store production data in DBFS root.**

Instead use governed storage through:

```text
Unity Catalog
```

and appropriate:

```text
External locations
Volumes
Managed tables
External tables
```

depending on the architecture.

---

# 11. Environment Separation

A production architecture should separate environments logically.

Typical model:

```text
DEV
 ↓
TEST / QA
 ↓
STAGING
 ↓
PRODUCTION
```

Each environment should have controlled:

```text
Code
Parameters
Data
Permissions
Compute
Secrets
Jobs
```

---

# 12. Do Not Hard-Code Environments

Bad:

```python
table = "prod.gold.customer"
```

Better:

```text
environment = dev/test/prod
```

and parameterize the relevant configuration.

Example:

```text
Job parameter:
environment = prod
```

Then:

```text
{{job.parameters.environment}}
```

can be referenced where supported.

---

# 13. Parameterized Production Jobs

A production Job should ideally be reusable.

Example:

```text
processing_date
environment
region
source_system
load_type
```

Instead of creating:

```text
Customer Job DEV
Customer Job QA
Customer Job PROD
```

with completely duplicated logic, use:

```text
Reusable Job
+
Environment-specific parameters/configuration
```

Current Databricks documentation supports Job parameters and dynamic value references for this purpose. :contentReference[oaicite:2]{index=2}

---

# 14. Job Parameters as Configuration

Example:

```text
Job Parameters

environment = prod
processing_date = 2026-08-17
region = us
```

Tasks consume these values.

Mental model:

```text
Job
 │
 ├── environment
 ├── processing_date
 └── region
       ↓
     Tasks
```

This reduces hard-coded values.

---

# 15. Parameterize Data Processing

Instead of:

```python
df = spark.read.table("prod.customer")
```

consider an architecture where the environment/table configuration is parameterized appropriately.

The exact implementation depends on the task type and security model.

### Principle

> **Separate workflow logic from environment-specific configuration.**

---

# 16. Parameter Security Trap

A Job parameter is **not a security mechanism**.

Current Databricks documentation explicitly notes that default parameter values are not a security control because users with sufficient run permissions can override Job parameters.

Therefore:

```text
Parameter
≠
Secret
```

Never use:

```text
password = myPassword123
```

as a Job parameter.

Use appropriate secret-management mechanisms instead.

---

# 17. Production DAG Design

A clean production DAG should represent business dependencies.

Example:

```text
                 Ingestion
                     │
                 Validation
                     │
              ┌──────┴──────┐
              ↓             ↓
        Customer Gold   Transaction Gold
              │             │
              └──────┬──────┘
                     ↓
                  Audit
```

This makes dependencies explicit.

---

# 18. Avoid One Giant Task

Bad architecture:

```text
One Notebook
  ├── Extract
  ├── Validate
  ├── Transform
  ├── Load
  ├── Audit
  └── Notify
```

Everything is hidden inside one task.

Problems:

```text
Poor observability
Difficult recovery
Hard to retry selectively
Poor dependency visibility
Large blast radius
```

---

# 19. Prefer Meaningful Task Boundaries

Better:

```text
Extract
  ↓
Validate
  ↓
Transform
  ↓
Load
  ↓
Audit
```

Benefits:

```text
Clear ownership
Targeted retry
Targeted repair
Better monitoring
Clear dependencies
Better failure isolation
```

---

# 20. But Don't Create Excessive Tasks

The opposite problem is:

```text
500 tiny tasks
```

This can create:

```text
Complex DAG
Management overhead
Scheduling overhead
Difficult troubleshooting
```

Use meaningful task boundaries.

### Principle

> **A task should represent a meaningful unit of work.**

---

# 21. Parallelize Independent Work

Suppose:

```text
Ingestion
   ↓
Validation
   ↓
       ┌──────────────┐
       ↓              ↓
Customer Gold   Product Gold
       │              │
       └──────┬───────┘
              ↓
            Audit
```

Customer and Product processing are independent.

They can potentially execute in parallel.

This reduces:

```text
Total Job duration
```

provided compute/source/target capacity supports it.

---

# 22. Don't Parallelize Dependent Work

Suppose:

```text
Silver
  ↓
Gold
```

Gold depends on Silver.

Running both simultaneously is incorrect.

Therefore:

```text
Dependency
→ correctness constraint
```

not merely:

```text
scheduling preference
```

---

# 23. Critical Data Validation

Production pipelines should validate critical data before allowing downstream processing.

Example:

```text
Ingestion
   ↓
Validation
   ↓
PASS?
 ┌─┴─┐
YES  NO
 ↓    ↓
Gold Alert/Block
```

The exact behavior depends on business requirements.

---

# 24. Business-Critical Validation

Not every validation failure should have the same response.

Example:

```text
99.95% records valid
```

may be acceptable for one dataset.

But:

```text
1 critical customer record corrupted
```

may require blocking.

Therefore:

> **Validation policy should consider business criticality, not only aggregate percentages.**

---

# 25. Warning vs Blocking

A production architecture may distinguish:

```text
WARNING
→ Continue downstream

CRITICAL FAILURE
→ Block affected downstream processing
```

Example:

```text
Customer validation
→ PASS

Transaction validation
→ CRITICAL FAIL

Customer Gold
→ Continue

Transaction Gold
→ Block
```

This is a business/architecture decision, not a universal Databricks rule.

---

# 26. Failure Isolation

Consider:

```text
Customer
   ↓
Customer Gold

Transaction
   ↓
Transaction Gold
```

If:

```text
Transaction validation fails
```

don't automatically block:

```text
Customer Gold
```

if Customer Gold is independent.

### Principle

> **Block the smallest affected downstream scope.**

---

# 27. Dependency-Aware Downstream Execution

Suppose:

```text
Customer
   ↓
Customer Gold
```

and:

```text
Customer Gold
   ↓
Customer Audit
```

If Customer fails:

```text
Customer Gold
→ blocked/skipped as appropriate

Customer Audit
→ cannot safely proceed if it depends on Customer Gold
```

Dependencies communicate this relationship to the workflow engine.

---

# 28. Independent Audit Pattern

Suppose:

```text
Customer
   ↓
Customer Gold

Audit Source
   ↓
Audit Report
```

Customer fails.

If Audit Source is independent:

```text
Customer Gold → BLOCK
Audit Report   → CONTINUE
```

This is often preferable to stopping the entire Job.

---

# 29. Control Flow Pattern

Use:

```text
Dependencies
+
Run If
+
If/else
```

to implement production control flow.

Current Databricks supports:

```text
Run If
If/else condition
For each
```

for workflow control. :contentReference[oaicite:3]{index=3}

---

# 30. Production Retry Pattern

A robust pattern:

```text
Task
 ↓
Transient failure?
 ↓
Retry
 ↓
Success?
 ├── YES → Continue
 └── NO → Alert / Recovery
```

For permanent failure:

```text
Task
 ↓
Permission error
 ↓
Fix permission
 ↓
Repair
```

---

# 31. Production Idempotency Pattern

Every recoverable production write should answer:

> **What happens if this task executes twice?**

Good patterns may include:

```text
MERGE with stable keys
Deterministic overwrite
Delete + insert for a well-defined partition
Deduplication
Transactional design
```

The correct pattern depends on the target data model.

---

# 32. Idempotency Example

Suppose:

```text
processing_date = 2026-08-17
```

A retry should not create:

```text
2 × records for 2026-08-17
```

Instead, design the write so that rerunning:

```text
2026-08-17
```

produces the same correct target state.

---

# 33. Backfill Pattern

Production systems need a way to correct historical data.

Normal processing:

```text
Today
 ↓
Incremental processing
```

Exceptional correction:

```text
Historical date
 ↓
Targeted backfill
 ↓
Validation
 ↓
Repair/reprocess
```

Do not make the normal daily pipeline perform full historical processing every day.

---

# 34. Incremental + Backfill Architecture

A robust pattern:

```text
              Production Pipeline
                     │
           ┌─────────┴─────────┐
           ↓                   ↓
      Normal Path         Exception Path
           ↓                   ↓
     Incremental          Targeted Backfill
           ↓                   ↓
           └─────────┬─────────┘
                     ↓
                 Validation
                     ↓
                   Gold
```

This balances:

```text
Efficiency
+
Historical correctness
```

---

# 35. CDC + Backfill

CDC is useful for incremental processing:

```text
Source
 ↓
CDC
 ↓
Changed records
 ↓
Target
```

But CDC does not prove:

```text
Result is correct
```

Therefore:

```text
CDC
→ tells you what changed

Validation
→ determines whether result is correct
```

For exceptional historical corrections:

```text
Targeted backfill
```

remains important.

---

# 36. Production Trigger Pattern

Choose the trigger based on business readiness.

### Time-based

```text
2 AM
 ↓
Scheduled Trigger
```

### File-driven

```text
File arrives
 ↓
File Arrival Trigger
```

### Table-driven

```text
Table updates
 ↓
Table Update Trigger
```

### Continuous workload

```text
Continuous Job
```

---

# 37. Trigger + Concurrency

Production trigger design must consider:

```text
Trigger frequency
+
Job runtime
+
Maximum concurrent runs
+
Queueing
```

Example:

```text
Schedule = every 15 min
Runtime = 45 min
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
Max concurrent runs = 1
```

and consider queueing.

---

# 38. Event-Driven Architecture

For irregular data arrival:

```text
File Arrival
     ↓
Job
     ↓
Incremental Processing
```

This can avoid unnecessary scheduled execution when no new data exists.

But remember:

```text
Trigger
→ decides WHEN

Processing logic
→ decides WHAT
```

---

# 39. Batch Completion Pattern

If files arrive in batches:

```text
File 1
File 2
File 3
...
File N
```

Use appropriate file-arrival trigger settings such as:

```text
Wait after last change
```

to debounce arrivals when the requirement is:

> Process after the batch has finished arriving.

---

# 40. Concurrency Pattern

Suppose:

```text
500 independent folders
```

A For Each task can process them with controlled concurrency.

Example:

```text
For Each
Inputs = 500
Concurrency = 50
```

Architecture:

```text
                  For Each
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Folder 1      Folder 2      Folder 3
       ...
                  Folder 500
```

Only the configured number of iterations execute concurrently.

---

# 41. Production Concurrency Rule

Never choose:

```text
Concurrency = 100
```

simply because it is allowed.

Choose based on:

```text
Measured performance
Source capacity
Target capacity
Compute
SLA
Cost
```

Example:

```text
10 → 3h
25 → 2h 15m
50 → 1h 45m
100 → 1h 40m
```

If SLA = 2h:

```text
50
```

may be preferable to 100.

---

# 42. Production Monitoring Pattern

Every critical Job should have:

```text
Run monitoring
Task monitoring
Failure alerts
Duration monitoring
SLA monitoring
Cost monitoring
```

Conceptually:

```text
Job
 ↓
Monitor
 ├── Status
 ├── Duration
 ├── Errors
 ├── Retries
 ├── Cost
 └── SLA
```

---

# 43. Actionable Notifications

Avoid alerting on everything.

Good:

```text
Critical Job failed
SLA exceeded
Streaming backlog high
Retries exhausted
```

Potentially noisy:

```text
Every successful task
Every retry attempt
Every normal run
```

unless the business specifically requires them.

---

# 44. Production Ownership

A production Job should have clear ownership.

Define:

```text
Business owner
Technical owner
On-call/support team
Recovery procedure
SLA
```

This is an architecture/operations best practice rather than a specific Lakeflow Jobs feature.

---

# 45. Git and Source Control

Production Jobs should use version-controlled code.

Current Lakeflow Jobs supports Git integration for supported task assets.

Conceptually:

```text
Developer
   ↓
Git
   ↓
Review
   ↓
Deploy
   ↓
Production Job
```

This provides:

```text
Version history
Code review
Rollback capability
Change traceability
```

---

# 46. Infrastructure as Code

For repeatable Job configuration, consider:

```text
Declarative Automation Bundles
```

or supported APIs/CLI.

Current Databricks documentation supports managing Jobs through:

```text
REST API
Databricks CLI
Declarative Automation Bundles
Databricks SDKs
```

This makes infrastructure more reproducible.

---

# 47. CI/CD Pattern

A production workflow can follow:

```text
Developer
    ↓
Git branch
    ↓
Pull Request
    ↓
Tests
    ↓
Deploy to DEV
    ↓
Validation
    ↓
Deploy to QA
    ↓
Approval
    ↓
Production
```

This reduces uncontrolled production changes.

---

# 48. Environment Promotion

Do not manually rebuild the Job from scratch in each environment.

Prefer:

```text
Same workflow definition
+
Environment-specific configuration
```

Example:

```text
DEV
environment = dev

QA
environment = qa

PROD
environment = prod
```

---

# 49. Secrets

Never put secrets directly into:

```text
Job parameters
Source code
Notebook code
Tags
Git repository
```

Use supported secret-management and authentication mechanisms.

### Professional principle

> **Configuration can be parameterized; secrets must be protected.**

---

# 50. Logging

Production tasks should provide enough information to answer:

```text
What happened?
Which input?
Which partition/date?
Which run?
Which task?
What failed?
```

Avoid logging:

```text
Passwords
Tokens
Secrets
Sensitive credentials
```

---

# 51. Auditability

Production workflows should allow you to determine:

```text
Who changed the Job?
Who ran it?
When did it run?
Which parameters were used?
Which code/version was executed?
What was the outcome?
```

Databricks provides:

```text
Job run information
System tables
Audit logs
Git integration
```

for different aspects of this problem.

---

# 52. Production Data Quality Pattern

A robust data pipeline:

```text
Source
  ↓
Ingestion
  ↓
Schema Validation
  ↓
Data Quality
  ↓
Transformation
  ↓
Business Validation
  ↓
Gold
  ↓
Audit
```

Do not rely only on:

```text
"Task completed successfully"
```

Technical execution success does not prove data correctness.

---

# 53. Technical Success vs Data Correctness

This is a major Professional concept.

A task can:

```text
SUCCESS
```

while producing:

```text
Incorrect data
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
Execution monitoring
+
Data validation
```

are both required.

---

# 54. Critical Validation Pattern

Example:

```text
Ingestion
   ↓
Validation
   ↓
┌───────────────┐
│ Critical?     │
└───────┬───────┘
        ↓
   ┌────┴────┐
   ↓         ↓
  PASS      FAIL
   ↓         ↓
  Gold     Block
             ↓
           Alert
```

---

# 55. Independent Work Pattern

Suppose:

```text
Customer
   ↓
Customer Gold

Transactions
   ↓
Transaction Gold
```

If:

```text
Transaction validation = FAILED
```

then:

```text
Customer Gold
→ Continue

Transaction Gold
→ Block
```

This minimizes the blast radius.

---

# 56. Blast Radius

A production architecture should minimize:

```text
Blast radius
```

If one dataset fails:

```text
Only dependent downstream processing
```

should ideally be affected.

Avoid:

```text
One small validation failure
        ↓
Entire enterprise pipeline stops
```

unless the business requirement genuinely requires it.

---

# 57. Critical Path + Blast Radius

Two important architecture goals:

```text
Critical Path
→ Minimize unnecessary runtime

Blast Radius
→ Minimize unnecessary failure propagation
```

Good DAG design addresses both.

---

# 58. Production Recovery Pattern

A production recovery flow:

```text
Failure
   ↓
Detect
   ↓
Notify
   ↓
Identify root cause
   ↓
Fix
   ↓
Check retry safety
   ↓
Repair targeted work
   ↓
Validate
   ↓
Resume downstream
```

---

# 59. Permanent Failure Pattern

Example:

```text
Permission denied
```

Correct:

```text
Failure
 ↓
Identify permission issue
 ↓
Fix permission
 ↓
Verify
 ↓
Repair
```

Incorrect:

```text
Permission denied
 ↓
Retry × 10
```

---

# 60. Transient Failure Pattern

Example:

```text
Temporary network failure
```

Correct:

```text
Failure
 ↓
Retry
 ↓
Backoff
 ↓
Success
```

provided the operation is retry-safe.

---

# 61. Production Idempotency Pattern

For every critical write:

```text
Can I execute this twice safely?
```

If:

```text
YES
```

recovery is simpler.

If:

```text
NO
```

design a safe recovery mechanism before enabling aggressive retry/repair.

---

# 62. Production Backfill Pattern

Normal path:

```text
Daily Incremental
```

Exception:

```text
Historical correction
```

Architecture:

```text
Normal
 ↓
Incremental

Exception
 ↓
Targeted Backfill
```

Don't turn:

```text
Daily incremental
```

into:

```text
Daily full reload
```

just to support rare historical corrections.

---

# 63. Production Streaming Pattern

Current Databricks production guidance for Structured Streaming recommends:

```text
Lakeflow Jobs
+
Continuous scheduling
+
Jobs compute
```

and recommends against using all-purpose compute for production streaming workloads.

Current guidance also states:

```text
Do not enable autoscaling for Structured Streaming jobs
```

for the recommended production configuration.

Official documentation:

https://docs.databricks.com/aws/en/structured-streaming/production

---

# 64. Streaming Architecture

Conceptually:

```text
Streaming Source
      ↓
Structured Streaming
      ↓
Checkpoint
      ↓
Target
```

or through a production orchestration model:

```text
Lakeflow Job
      ↓
Streaming Task
      ↓
Target
```

The exact architecture depends on the workload and whether Lakeflow Pipelines is appropriate.

---

# 65. Streaming Trap

Do not confuse:

```text
Continuous Job scheduling
```

with:

```text
Structured Streaming trigger interval
```

They are different concepts.

Current Databricks documentation explicitly distinguishes the two.

---

# 66. dbt Production Pattern

For production dbt transformations, Databricks recommends using:

```text
dbt task
```

inside a Databricks Job.

Conceptually:

```text
Lakeflow Job
    ↓
dbt Task
    ↓
dbt transformations
```

Current documentation recommends:

```text
dbt-databricks
```

rather than:

```text
dbt-spark
```

for Databricks dbt projects.

---

# 67. Production Architecture: Complete Example

Consider:

```text
Source Files
     ↓
File Arrival Trigger
     ↓
Ingestion
     ↓
Validation
     ↓
      ┌──────────────┐
      ↓              ↓
Customer Gold   Transaction Gold
      │              │
      └──────┬───────┘
             ↓
           Audit
             ↓
          Notify
```

Production controls:

```text
Service Principal
Unity Catalog
Serverless
Parameters
Controlled concurrency
Retry
Idempotent writes
Monitoring
Notifications
Repair
Backfill mechanism
```

---

# 68. Complete Production Pattern

```text
                         TRIGGER
                            ↓
                    Lakeflow Job
                            ↓
                    Service Principal
                            ↓
                       Parameters
                            ↓
                       Ingestion
                            ↓
                       Validation
                       /         \
                    PASS          FAIL
                     ↓              ↓
                 Transform       Alert/Block
                  /     \
                 ↓       ↓
            Customer   Transaction
              Gold       Gold
                 \       /
                  \     /
                    Audit
                      ↓
                 Monitoring
                      ↓
                 Notification
                      ↓
             Repair / Backfill
```

---

# 69. Production Design Checklist

Before deploying a Job, ask:

```text
□ Is the workflow correctly modeled as a DAG?

□ Are dependencies explicit?

□ Are independent tasks parallelized?

□ Are critical validations present?

□ Are downstream dependencies protected?

□ Is the Job parameterized?

□ Are secrets protected?

□ Is the production identity a service principal?

□ Is Unity Catalog used appropriately?

□ Is serverless appropriate?

□ If classic compute is required, is Jobs compute used?

□ Is concurrency tested?

□ Are overlapping runs safe?

□ Is queueing configured appropriately?

□ Are writes idempotent?

□ Are retries appropriate?

□ Is repair possible?

□ Is historical backfill possible?

□ Are monitoring and notifications configured?

□ Is SLA monitored?

□ Is cost monitored?

□ Is code version-controlled?

□ Is deployment reproducible?
```

---

# 70. Professional Scenario Framework

When a question asks:

> "Which architecture is best?"

Use this sequence:

```text
1. Understand the business requirement
        ↓
2. Identify data dependencies
        ↓
3. Identify critical validations
        ↓
4. Separate independent work
        ↓
5. Choose trigger
        ↓
6. Choose compute
        ↓
7. Control concurrency
        ↓
8. Design retries
        ↓
9. Ensure idempotency
        ↓
10. Design recovery/repair
        ↓
11. Add monitoring
        ↓
12. Add actionable notifications
        ↓
13. Consider security
        ↓
14. Consider cost/SLA
```

---

# 71. Exam Decision Rules

### Rule 1

> **Use Lakeflow Jobs for Databricks-native workflow orchestration whenever appropriate.**

### Rule 2

> **Use service principals for production Job execution.**

### Rule 3

> **Use least-privilege permissions.**

### Rule 4

> **Use Unity Catalog-compatible compute.**

### Rule 5

> **Prefer serverless for supported Job workloads.**

### Rule 6

> **For classic compute, use Jobs compute rather than all-purpose compute for production Jobs.**

### Rule 7

> **Separate independent tasks so they can run independently when safe.**

### Rule 8

> **Block only the affected downstream scope when possible.**

### Rule 9

> **Use validation to determine data correctness; successful task execution alone is insufficient.**

### Rule 10

> **Use retries for appropriate transient failures, not as a substitute for fixing deterministic failures.**

### Rule 11

> **Design writes to be idempotent when retries/repairs are expected.**

### Rule 12

> **Maintain a targeted backfill mechanism for exceptional historical corrections.**

### Rule 13

> **Use parameters instead of hard-coding environment-specific values.**

### Rule 14

> **Do not use Job parameters as a secret/security mechanism.**

### Rule 15

> **Choose concurrency based on measured SLA, capacity, correctness, and cost.**

### Rule 16

> **Monitor both technical execution and data correctness.**

### Rule 17

> **Minimize blast radius by modeling dependencies accurately.**

---

# 72. Common Exam Traps

## Trap 1

> Run production Jobs as a developer account.

❌ Poor production design.

Prefer:

```text
Service Principal
```

---

## Trap 2

> One failed dataset should stop every downstream dataset.

❌ Not necessarily.

First determine:

```text
Dependencies
Business criticality
Independent branches
```

---

## Trap 3

> 99.5% validation means always continue.

❌ Not necessarily.

Criticality may matter more than aggregate percentage.

---

## Trap 4

> CDC guarantees correct data.

❌ Incorrect.

```text
CDC
→ identifies changes

Validation
→ verifies correctness
```

---

## Trap 5

> Retry solves permission errors.

❌ Incorrect.

Fix the permission issue first.

---

## Trap 6

> Every historical correction should use a full reload.

❌ Usually inefficient.

Use:

```text
Targeted backfill
```

when appropriate.

---

## Trap 7

> More concurrency is always better.

❌ Incorrect.

Check:

```text
SLA
Source
Target
Compute
Cost
Correctness
```

---

## Trap 8

> Successful task = correct data.

❌ Incorrect.

Technical execution and data correctness are separate concerns.

---

## Trap 9

> Job parameters are secure because users cannot edit the Job definition.

❌ Incorrect.

Users with appropriate run permissions can override Job parameter values.

---

## Trap 10

> Serverless should be forced onto every workload.

❌ Incorrect.

Use it when the workload is supported and appropriate.

---

## Trap 11

> Continuous Job scheduling = Structured Streaming trigger interval.

❌ Incorrect.

These are separate concepts.

---

## Trap 12

> Production streaming should use all-purpose compute.

❌ Current Databricks production guidance recommends Jobs compute and continuous Lakeflow Job scheduling for production Structured Streaming workloads.

---

# 73. Quick Reference

| Production concern | Pattern |
|---|---|
| Orchestration | Lakeflow Jobs |
| Production identity | Service principal |
| Governance | Unity Catalog |
| Supported compute | Serverless |
| Unsupported serverless workload | Classic Jobs compute |
| Production classic workload | Jobs compute |
| Configuration | Job parameters |
| Secrets | Secret-management mechanism |
| Dependencies | Explicit DAG |
| Independent work | Parallelize when safe |
| Validation | Gate critical downstream processing |
| Transient failure | Retry |
| Permanent failure | Fix + repair |
| Retry safety | Idempotent writes |
| Historical correction | Targeted backfill |
| Event-driven input | Event trigger |
| Time-based input | Scheduled trigger |
| High-volume iteration | For Each + tested concurrency |
| Run overlap | Max concurrent runs |
| Missed runs | Queueing where appropriate |
| Monitoring | Jobs UI + system tables |
| Alerting | Actionable notifications |
| Long-term analytics | `system.lakeflow` |
| Cost analysis | Billing + system tables |
| Version control | Git |
| Reproducible deployment | Bundles/API/CLI |

---

# 74. Final Production Mental Model

Memorize:

```text
                 PRODUCTION JOB
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    SECURITY       CORRECTNESS    RELIABILITY
        │              │              │
 Service Principal  Validation      Retry
 Least Privilege   Idempotency      Repair
 Unity Catalog     Backfill         Recovery
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                  ORCHESTRATION
                       │
              Dependencies / Flow
                       ↓
                  PERFORMANCE
                       │
               Compute / Parallelism
                       ↓
                     SLA
                       │
                      COST
                       ↓
                  MONITORING
                       │
                 Notifications
```

---

# 75. One-Line Rules to Memorize

```text
Production identity
→ Service principal.

Production governance
→ Unity Catalog.

Supported Job workload
→ Prefer serverless.

Classic production workload
→ Jobs compute, not all-purpose.

DAG
→ Model real business dependencies.

Independent branches
→ Don't unnecessarily block them.

Validation
→ Proves data correctness, not just execution success.

CDC
→ Identifies changes, not correctness.

Retry
→ Transient + retry-safe.

Permanent failure
→ Fix cause first.

Repair
→ Targeted recovery.

Idempotency
→ Safe repeated execution.

Backfill
→ Exceptional historical correction mechanism.

Parameters
→ Reusable/configurable workflow.

Secrets
→ Never treat parameters as security controls.

Concurrency
→ Choose using measured performance and capacity.

Monitoring
→ Technical execution + data correctness.

Notifications
→ Actionable, not noisy.

Blast radius
→ Keep failure scope as small as safely possible.
```

---

# 76. Official Documentation

Use these as the primary references for this chapter:

- [Production job scheduling cheat sheet](https://docs.databricks.com/aws/en/cheat-sheet/jobs)
- [Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/)
- [Manage identities, permissions, and privileges for Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/privileges)
- [Configure compute for jobs](https://docs.databricks.com/aws/en/jobs/compute)
- [Configure and edit Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/configure-job)
- [Parameterize jobs](https://docs.databricks.com/aws/en/jobs/parameters)
- [Dynamic value references](https://docs.databricks.com/aws/en/jobs/dynamic-value-references)
- [Configure task dependencies](https://docs.databricks.com/aws/en/jobs/run-if)
- [Production considerations for Structured Streaming](https://docs.databricks.com/aws/en/structured-streaming/production)
- [Use dbt transformations in Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/how-to/use-dbt-in-workflows)

---

## Chapter Status

**10. Production Architecture Patterns — COMPLETE ✅**

### Key verified facts

```text
Production orchestration
→ Lakeflow Jobs for Databricks-native workflows

Production identity
→ Service principal recommended

Governance
→ Unity Catalog-compatible compute

Serverless
→ Recommended for most supported Job workloads

Classic compute
→ Jobs compute preferred over all-purpose compute

Parameters
→ Support reusable/configurable workflows

Job parameters
→ Not a security control

Concurrency
→ Must be designed with source/target/capacity/SLA/cost

Retries
→ Should be appropriate and retry-safe

Repair
→ Targeted recovery

Validation
→ Separate from technical execution success

System tables
→ Useful for historical/account-level monitoring and cost analysis

Production streaming
→ Current Databricks guidance recommends Lakeflow Jobs with continuous scheduling and Jobs compute
```

