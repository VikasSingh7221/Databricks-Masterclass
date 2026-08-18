# 04. Parameters & Dynamic Values

> **Databricks Data Engineer Professional — Certification Notes**

---

# 1. Why Parameters Matter

Parameters allow a Lakeflow Job to be **reusable and configurable** instead of hard-coding values into individual tasks.

Instead of:

```text
environment = "prod"
processing_date = "2026-08-17"
```

inside code, the workflow can receive:

```text
environment
processing_date
batch_size
```

as configuration.

### Core mental model

```text
Configuration
      ↓
Parameters
      ↓
Tasks
```

---

# 2. Parameter Concepts

The main concepts to distinguish are:

```text
Job Parameters
Task Parameters
Dynamic Value References
Task Values
```

They solve different problems.

| Concept | Purpose |
|---|---|
| Job Parameter | Job/run-level configuration |
| Task Parameter | Input/configuration for a specific task |
| Dynamic Value Reference | Reference runtime metadata or values |
| Task Value | Runtime-generated value produced by a task |

---

# 3. Job Parameters

A **Job Parameter** is defined at the Job level and can be made available to tasks.

Example:

```text
environment = prod
processing_date = 2026-08-17
batch_size = 5000
```

These values represent configuration for the Job.

### Typical examples

```text
environment
processing_date
region
source_system
batch_size
load_type
```

---

# 4. Why Use Job Parameters?

Without parameters:

```python
environment = "prod"
```

The code is tightly coupled to one environment.

With parameters:

```text
environment = dev
environment = qa
environment = prod
```

the same Job can be reused.

### Benefits

- Reusability
- Environment-specific execution
- Easier automation
- Less hard-coding
- Easier manual reruns
- Better operational flexibility

---

# 5. Example — Processing Date

Instead of:

```python
processing_date = "2026-08-17"
```

use a Job Parameter:

```text
processing_date
```

Then a run can receive:

```text
processing_date = 2026-08-17
```

Another run:

```text
processing_date = 2026-08-16
```

This is especially useful for:

- Backfills
- Reruns
- Historical processing
- Daily pipelines

---

# 6. Job Parameter Override

A Job Parameter can have a default value.

For example:

```text
environment = dev
```

A specific Job Run can override it:

```text
environment = prod
```

This allows the same Job definition to support different execution contexts.

---

# 7. Task Parameters

Task parameters provide input to a particular task.

Example:

```text
Task: Load Customer Data

Parameters:
target_table = customer
load_type = incremental
```

The task can use these values during execution.

### Mental model

```text
Job Parameters
      ↓
Job-level configuration

Task Parameters
      ↓
Task-specific configuration
```

---

# 8. Job Parameter vs Task Parameter

This distinction is a common Professional exam topic.

### Job Parameter

```text
environment = prod
```

is Job-level configuration.

### Task Parameter

```text
target_table = customer
```

is task-specific configuration.

---

# 9. Parameter Precedence

This is a **high-priority exam rule**.

For tasks that accept key-value parameters, when the same key exists at both levels:

```text
Job Parameter
      ↓
Task Parameter
```

the **Job Parameter takes precedence**.

### Example

Job:

```text
environment = prod
```

Task:

```text
environment = qa
```

Same key:

```text
environment
```

The task receives:

```text
prod
```

### Memorize

> **Same key + key-value parameter task → Job Parameter wins.**

Official documentation:

https://docs.databricks.com/aws/en/jobs/job-parameters

---

# 10. Why Parameter Precedence Matters

Suppose a task has:

```text
environment = qa
```

but the Job Run is launched with:

```text
environment = prod
```

The Job-level value can control the execution.

This is useful when operationally overriding task configuration without modifying the task definition.

---

# 11. Important Scope Distinction

Do not confuse:

```text
Job Parameter
```

with:

```text
Task Value
```

A Job Parameter is normally an **input/configuration**.

A Task Value is a **runtime output**.

### Compare

```text
Job Parameter
    ↓
Known before/during Job execution
    ↓
Task consumes it
```

versus:

```text
Task A executes
    ↓
calculates result
    ↓
Task Value
    ↓
Task B consumes result
```

---

# 12. Task Values

Task Values allow a task to publish a runtime-generated value that another task can reference.

Example:

```python
row_count = 125000

dbutils.jobs.taskValues.set(
    key="row_count",
    value=row_count
)
```

The task has now produced:

```text
row_count = 125000
```

---

# 13. When to Use Task Values

Use Task Values for small runtime-generated pieces of information.

Examples:

```text
row_count
max_date
validation_status
generated_path
folder_list
record_count
```

Example:

```text
Task A
   ↓
calculates max_date
   ↓
Task Value
   ↓
Task B
```

---

# 14. Task Value Example

Python:

```python
from datetime import date

max_date = "2026-08-17"

dbutils.jobs.taskValues.set(
    key="max_date",
    value=max_date
)
```

Another task can reference:

```text
{{tasks.task_a.values.max_date}}
```

Conceptually:

```text
Task A
  │
  │ Task Value
  ↓
max_date
  │
  │ Dynamic Reference
  ↓
Task B
```

---

# 15. Dynamic Value References

A **dynamic value reference** is an expression used to access runtime values.

Examples:

```text
{{job.parameters.environment}}
```

```text
{{tasks.task_a.values.row_count}}
```

```text
{{job.run_id}}
```

```text
{{task.run_id}}
```

Dynamic references allow task configuration to depend on runtime context.

Official documentation:

https://docs.databricks.com/aws/en/jobs/dynamic-value-references

---

# 16. Task Value vs Dynamic Value Reference

This distinction is important.

### Task Value

The value is created:

```text
Task A
 ↓
Task Value
```

### Dynamic Value Reference

The syntax used to retrieve/reference the value:

```text
{{tasks.task_a.values.row_count}}
```

Therefore:

```text
Task Value
→ WHAT is produced

Dynamic Reference
→ HOW another task accesses it
```

---

# 17. Job Metadata Dynamic References

Dynamic references can access Job and task metadata.

Examples include:

```text
{{job.id}}
{{job.run_id}}
{{job.parameters.<key>}}
{{task.name}}
{{task.run_id}}
```

The exact set of supported references should be checked against current Databricks documentation.

---

# 18. Why Dynamic References Are Useful

They allow configuration to change automatically based on the current execution.

Example:

```text
Job Run
   ↓
run_id = 12345
   ↓
Task
   ↓
output path includes run_id
```

Instead of hard-coding:

```text
/run/12345
```

you can reference the runtime Job information.

---

# 19. Passing Runtime Values Between Tasks

Example:

```text
Generate Batch
      ↓
Task Value:
batch_id = 7821
      ↓
Process Batch
```

The downstream task references:

```text
{{tasks.generate_batch.values.batch_id}}
```

This enables runtime dependency passing.

---

# 20. Task Values and For Each

This is an important architecture pattern.

Suppose Task A discovers:

```text
500 folders
```

Task A can store the folder list as a Task Value.

```python
dbutils.jobs.taskValues.set(
    key="folders",
    value=folders
)
```

Then the For Each task can use:

```text
{{tasks.generate_folders.values.folders}}
```

as its input.

Architecture:

```text
Generate Folders
       ↓
Task Value
       ↓
Dynamic Reference
       ↓
For Each
       ↓
500 iterations
```

---

# 21. Task Value Limits

Task Values are intended for passing values between tasks, not for storing large datasets.

Current Databricks documentation specifies:

- Up to **250 task values per Job Run**
- Each Task Value's JSON representation is limited to **48 KiB**

Official documentation:

https://docs.databricks.com/aws/en/dev-tools/databricks-utils

### Important architecture rule

Do not try to pass a massive dataset through Task Values.

Bad:

```text
1 million records
      ↓
Task Value
```

Better:

```text
1 million records
      ↓
Delta / cloud storage
      ↓
Pass table/path/reference
```

---

# 22. Parameters vs Data

A useful rule:

### Small configuration

Use:

```text
Job Parameter
```

### Small runtime result

Use:

```text
Task Value
```

### Large dataset

Use:

```text
Delta table
Cloud storage
Volume
External data store
```

Do not use Job Parameters or Task Values as a substitute for a data storage layer.

---

# 23. Parameters + Incremental Processing

Suppose:

```text
processing_date = 2026-08-17
```

The Job can use this parameter to determine what data to process.

Example:

```text
Job Parameter
      ↓
processing_date
      ↓
Filter source data
      ↓
Process only required partition/date
```

This supports reusable incremental pipelines.

---

# 24. Parameters + Backfill

Suppose the normal pipeline processes:

```text
processing_date = today
```

A historical correction is needed for:

```text
2026-08-01
```

Instead of changing the Job definition:

```text
Run Job
processing_date = 2026-08-01
```

This allows the same pipeline to perform a targeted backfill.

### Mental model

```text
Normal run
→ today's date

Backfill
→ historical date passed as parameter
```

---

# 25. Parameters + Environment

A reusable Job might have:

```text
environment
catalog
schema
processing_date
```

Example:

```text
environment = prod
catalog = prod_catalog
schema = gold
processing_date = 2026-08-17
```

The same workflow definition can be adapted across environments using configuration rather than code changes.

---

# 26. Parameters + For Each

Suppose:

```text
source_system = Salesforce
```

and:

```text
folders = [...]
```

The Job can combine:

```text
Job Parameters
      +
For Each input
      ↓
Task execution
```

Example conceptual input:

```text
environment = prod
folder = customer_001
```

The environment may come from Job configuration while the folder comes from the For Each iteration.

---

# 27. Dynamic References in Control Flow

Dynamic values can also be useful for conditional workflow behavior.

Example:

```text
row_count > 0
```

Conceptually:

```text
Task A
  ↓
row_count
  ↓
Conditional evaluation
  ↓
TRUE / FALSE
```

This allows workflows to branch based on runtime information.

---

# 28. Parameterizing SQL

Suppose a SQL task needs:

```text
processing_date
```

Instead of hard-coding:

```sql
WHERE order_date = '2026-08-17'
```

parameterize the task appropriately.

Conceptually:

```text
Job Parameter
      ↓
processing_date
      ↓
SQL Task
      ↓
WHERE order_date = processing_date
```

The exact syntax depends on the task type and parameter mechanism being used.

---

# 29. Parameterizing Notebook Tasks

A Notebook task can consume configured parameters.

Conceptually:

```text
Job Parameter
      ↓
Notebook Task
      ↓
Notebook code
```

The notebook reads the supplied parameter rather than hard-coding configuration.

This makes the notebook reusable.

---

# 30. Parameterization vs Hard-Coding

### Bad

```python
environment = "prod"
schema = "gold"
processing_date = "2026-08-17"
```

### Better

```text
environment
schema
processing_date
```

provided through Job/task configuration.

### Benefits

- Reusability
- Easier testing
- Easier deployment
- Easier backfill
- Less code modification
- Better operational control

---

# 31. Parameter Precedence — Exam Scenario

Question:

```text
Job:
environment = prod

Task:
environment = qa
```

What value does the task receive?

Answer:

```text
prod
```

because the Job Parameter takes precedence for the relevant key-value parameter task.

### Remember

```text
Same key
    ↓
Job Parameter wins
```

Do not memorize the opposite.

---

# 32. Task Values — Exam Scenario

Question:

> Task A discovers the maximum processed timestamp during execution. Task B needs that timestamp.

Best mechanism:

```text
Task A
   ↓
Task Value
   ↓
Dynamic Value Reference
   ↓
Task B
```

Not:

```text
Job Parameter
```

because the value is generated dynamically during execution.

---

# 33. Job Parameter — Exam Scenario

Question:

> The same pipeline should run for different processing dates without changing code.

Best mechanism:

```text
Job Parameter
```

Example:

```text
processing_date = 2026-08-17
```

---

# 34. Task Parameter — Exam Scenario

Question:

> Only one task needs a specific configurable target table.

Use task-specific configuration/parameterization rather than unnecessarily making the value a global Job Parameter.

Example:

```text
Task:
Load Customer Gold

Parameter:
target_table = customer_gold
```

---

# 35. Dynamic Reference — Exam Scenario

Question:

> Task B needs the value generated by Task A.

Use:

```text
Task Value
+
Dynamic Value Reference
```

Example:

```text
{{tasks.task_a.values.row_count}}
```

---

# 36. What NOT to Use

### Don't use Job Parameters for:

```text
Large datasets
Millions of records
Runtime-generated bulk data
```

---

### Don't use Task Values for:

```text
Large datasets
Persistent storage
Long-term state
```

---

### Don't hard-code:

```text
environment
processing_date
catalog
schema
batch configuration
```

when they should be configurable.

---

# 37. Parameter Architecture

A production workflow may look like:

```text
                 Job
                  │
           Job Parameters
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
 environment  date       batch_size
       │          │          │
       └──────────┼──────────┘
                  ↓
                Tasks
                  │
                  ↓
              Task Values
                  │
                  ↓
         Dynamic References
                  │
                  ↓
            Downstream Tasks
```

---

# 38. Complete Example

Suppose a nightly pipeline processes customer folders.

### Job Parameters

```text
environment = prod
processing_date = 2026-08-17
```

### Task A

Discovers:

```text
folders = [
    "customer_001",
    "customer_002",
    "customer_003"
]
```

Stores:

```python
dbutils.jobs.taskValues.set(
    key="folders",
    value=folders
)
```

### For Each

Input:

```text
{{tasks.discover_folders.values.folders}}
```

### Nested Task

Processes:

```text
{{input}}
```

Architecture:

```text
Job Parameters
      ↓
Discover Folders
      ↓
Task Value: folders
      ↓
Dynamic Reference
      ↓
For Each
      ↓
Customer folders
```

---

# 39. Parameter Decision Framework

When you encounter a parameter question, ask:

```text
1. Is the value configuration for the Job?
       ↓
   Job Parameter

2. Is it configuration specific to one task?
       ↓
   Task Parameter

3. Is it generated during execution?
       ↓
   Task Value

4. Does another task need to reference it?
       ↓
   Dynamic Value Reference

5. Is the value a large dataset?
       ↓
   Store it in a data system instead
```

---

# 40. Professional Exam Decision Rules

### Rule 1

> **Use Job Parameters for reusable Job/run configuration.**

### Rule 2

> **Use Task Parameters for task-specific configuration.**

### Rule 3

> **When the same key is supplied through Job and task key-value parameters, the Job Parameter takes precedence.**

### Rule 4

> **Use Task Values for runtime-generated values.**

### Rule 5

> **Use Dynamic Value References to consume runtime values and metadata.**

### Rule 6

> **Do not use parameters as a data-storage mechanism.**

### Rule 7

> **Use parameters to make pipelines reusable across dates, environments, and controlled backfills.**

### Rule 8

> **Use Task Values + Dynamic References when one task produces a runtime value required by another task.**

---

# 41. Common Exam Traps

## Trap 1 — Parameter precedence

```text
Job:
environment = prod

Task:
environment = qa
```

Wrong:

```text
qa
```

Correct:

```text
prod
```

---

## Trap 2 — Task Value vs Job Parameter

Question:

> A value is discovered during execution.

Wrong:

```text
Job Parameter
```

Correct:

```text
Task Value
```

---

## Trap 3 — Task Value vs Dynamic Reference

Remember:

```text
Task Value
→ creates/stores runtime value

Dynamic Reference
→ references the value
```

---

## Trap 4 — Large data through Task Value

Don't pass:

```text
100 MB dataset
```

through Task Values.

Use storage and pass a reference.

---

## Trap 5 — Hard-coded backfill

Don't modify code every time you need:

```text
processing_date = yesterday
```

or:

```text
processing_date = 30 days ago
```

Parameterize the processing date.

---

# 42. Quick Reference Table

| Requirement | Use |
|---|---|
| Environment configuration | Job Parameter |
| Processing date | Job Parameter |
| Batch size | Job Parameter |
| Task-specific target | Task Parameter |
| Runtime row count | Task Value |
| Runtime generated path | Task Value |
| Runtime max timestamp | Task Value |
| Access Task Value | Dynamic Value Reference |
| Access Job parameter | Dynamic Value Reference |
| Large dataset | Delta/storage |
| Historical rerun | Processing-date parameter |
| Dynamic For Each input | Task Value + Dynamic Reference |

---

# 43. Final Mental Model

```text
                 INPUT
                   │
          ┌────────┴────────┐
          ↓                 ↓
     Job Parameter      Task Parameter
          │                 │
          └────────┬────────┘
                   ↓
                 TASK
                   │
              executes
                   ↓
             Runtime result
                   │
                   ↓
              Task Value
                   │
                   ↓
        Dynamic Value Reference
                   │
                   ↓
           Downstream Task
```

Remember:

```text
Job Parameter
→ configuration

Task Parameter
→ task-specific configuration

Task Value
→ runtime output

Dynamic Reference
→ runtime lookup/reference
```

---

# 44. Certification Summary

For Professional-level questions, focus on the **reason behind the mechanism**:

```text
Need reusable configuration?
→ Job Parameter

Need task-specific configuration?
→ Task Parameter

Need runtime-generated information?
→ Task Value

Need to consume runtime information?
→ Dynamic Value Reference

Need historical processing?
→ Parameterize processing date

Need environment portability?
→ Parameterize environment/catalog/schema

Need many runtime inputs?
→ Task Value + Dynamic Reference + For Each

Need large data?
→ Data storage, not Task Values
```

---

# 45. Official Documentation

For certification preparation, verify current behavior against Databricks documentation:

- [Lakeflow Jobs parameters](https://docs.databricks.com/aws/en/jobs/parameters)
- [Job parameters](https://docs.databricks.com/aws/en/jobs/job-parameters)
- [Dynamic value references](https://docs.databricks.com/aws/en/jobs/dynamic-value-references)
- [Task values](https://docs.databricks.com/aws/en/dev-tools/databricks-utils)
- [For Each task](https://docs.databricks.com/aws/en/jobs/tasks/for-each)

---
