# 02. Tasks & Task Dependencies

> **Databricks Data Engineer Professional — Certification Notes**

---

# 1. What is a Task?

A **task** is an individual unit of work inside a Lakeflow Job.

Examples:

```text
Ingest Customers
Validate Customers
Transform Customers
Load Gold
Send Notification
```

A Job can contain multiple tasks.

```text
Job
│
├── Ingestion
├── Validation
├── Transformation
└── Audit
```

### Mental model

> **Job = workflow**

> **Task = unit of work**

---

# 2. Task Types

Lakeflow Jobs supports multiple task types.

Common examples include:

- Notebook task
- Python script task
- Python wheel task
- SQL task
- Pipeline task
- Run Job task
- For Each task
- If/else condition
- Other supported task types

The exact task types available can change with the Databricks product/version, so use the current Databricks documentation when a certification question asks about a specific task type.

---

# 3. Why Task Dependencies Matter

A dependency defines the relationship between tasks.

Example:

```text
A → B
```

means:

> Task B depends on Task A.

The dependency establishes the execution relationship.

---

# 4. Simple Dependency Chain

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

Execution:

```text
Extract
   ↓
Validate
   ↓
Transform
   ↓
Load
```

This is appropriate when each task requires the output or successful completion of the previous task.

---

# 5. Parallel Dependencies

Suppose we have:

```text
Salesforce Ingestion ──┐
                       ├──→ Validation
Snowflake Ingestion ───┘
```

Salesforce and Snowflake ingestion are independent.

They can potentially execute in parallel:

```text
Salesforce ────────────┐
                       ↓
                    Validation
                       ↑
Snowflake ─────────────┘
```

### Professional principle

> **Only create a dependency when there is a real execution/data/business dependency.**

---

# 6. Unnecessary Dependencies

Suppose:

```text
Task A
   ↓
Task B
   ↓
Task C
```

But in reality:

```text
A and B are independent
C requires both
```

Then the better design is:

```text
A ──┐
    ├──→ C
B ──┘
```

Why?

Because unnecessary dependencies serialize work.

### Bad

```text
A → B → C
```

### Better

```text
A ──┐
    ├──→ C
B ──┘
```

This can improve:

- Runtime
- Throughput
- SLA
- Resource utilization

---

# 7. Dependency vs Execution Order

A dependency should not be added simply because:

> "I want B to run after A."

Ask:

> **Does B actually require A?**

If yes:

```text
A → B
```

If no:

```text
A ──┐
    ├──→ C
B ──┘
```

### Professional exam principle

> **Model the real dependency graph, not an artificial sequential order.**

---

# 8. Upstream and Downstream

Consider:

```text
A → B → C
```

From B's perspective:

```text
A = upstream
C = downstream
```

From A's perspective:

```text
B = downstream
```

From C's perspective:

```text
B = upstream
```

### Terminology

```text
Upstream
→ task(s) that a task depends on

Downstream
→ task(s) that depend on a task
```

---

# 9. Multiple Upstream Tasks

Example:

```text
       A
       │
       ├────→ C
       │
       B
```

C has two upstream tasks:

```text
A
B
```

The behavior of C depends on its configured dependency and `Run If` condition.

For example:

```text
C
Run If = ALL SUCCEEDED
```

means C requires all configured upstream tasks to succeed.

---

# 10. Multiple Downstream Tasks

Example:

```text
          B
         ↗
A ──────
         ↘
          C
```

Both B and C depend on A.

If B and C are independent of each other:

```text
A
├──→ B
└──→ C
```

They can potentially execute independently after A satisfies the required condition.

---

# 11. Dependency Graph Example

Consider a customer pipeline:

```text
                Customer Ingestion
                       │
                       ↓
                  Customer DQ
                       │
                ┌──────┴──────┐
                ↓             ↓
          Customer Gold    Audit Report
```

Here:

```text
Customer Ingestion
        ↓
Customer DQ
```

is a direct dependency.

But after DQ succeeds:

```text
Customer Gold
Audit Report
```

can be independent downstream branches if they don't depend on each other.

---

# 12. Critical vs Independent Dependencies

This distinction is extremely important.

Suppose:

```text
Customer Ingestion
        ↓
Customer Validation
        ↓
Customer Gold
```

Customer Gold requires valid Customer data.

Therefore:

```text
Validation failure
       ↓
Customer Gold blocked
```

But suppose:

```text
Customer Audit Report
```

depends only on the successfully ingested Customer dataset and has its own appropriate validation dependency.

Then it may be possible for the Audit Report to proceed even if another unrelated downstream task is blocked.

### Professional principle

> **Failure propagation should follow actual dependencies and business requirements.**

---

# 13. Dependency Does Not Mean Success

A dependency defines a relationship.

It does **not automatically mean**:

> "The downstream task always runs after the upstream task."

The downstream task may be affected by:

- Upstream success
- Upstream failure
- Upstream exclusion
- Cancellation
- `Run If` conditions
- Other control-flow rules

Therefore:

```text
Dependency
    +
Run If
    ↓
Actual downstream behavior
```

---

# 14. Dependency vs Run If

These are frequently confused.

## Dependency

Answers:

> **Which tasks does this task depend on?**

Example:

```text
A → B
```

---

## Run If

Answers:

> **Given the upstream outcomes, should B execute?**

Example:

```text
A
↓
B

B:
Run If = ALL SUCCEEDED
```

So:

```text
Dependency
→ relationship

Run If
→ condition
```

---

# 15. Example — Success Path

```text
A → B → C
```

If:

```text
A = SUCCESS
B = SUCCESS
```

then:

```text
C
```

can become eligible according to its configured condition.

---

# 16. Example — Failure Path

```text
A → B → C
```

Suppose:

```text
A = FAILED
```

Then B's configured `Run If` determines whether B can execute.

If B requires:

```text
ALL SUCCEEDED
```

B won't execute.

That can affect C as well.

Therefore, when analyzing a failed DAG, don't simply ask:

> "Which task failed?"

Also ask:

```text
What are the dependencies?
What is each Run If condition?
Which tasks became excluded/skipped?
Which downstream tasks still have a valid path?
```

---

# 17. Failure Isolation

Suppose:

```text
A ──→ B
C ──→ D
```

A fails.

If C/D are completely independent:

```text
A ❌ → B blocked

C ✅ → D can continue
```

This is **failure isolation**.

A failure should not unnecessarily stop unrelated work.

---

# 18. Bad Architecture — Global Dependency

Suppose:

```text
A → B → C → D → E
```

but the actual business dependencies are:

```text
A ──→ C
B ──→ D
C ──→ E
D ──→ E
```

The first design unnecessarily serializes the workflow.

Better:

```text
A ──→ C ──┐
           ├──→ E
B ──→ D ──┘
```

This allows independent branches to progress in parallel.

---

# 19. Dependency Design and Performance

Dependencies can directly affect runtime.

### Sequential

```text
A → B → C → D

Runtime:
10 + 10 + 10 + 10
≈ 40 minutes
```

### Parallel branches

```text
A ──┐
B ──┼──→ D
C ──┘
```

If A, B, and C can run independently:

```text
A = 10 min
B = 10 min
C = 10 min

Potential branch time ≈ 10 min
```

Then D executes.

### Important

This is only beneficial when:

- The tasks are actually independent
- Resources can support parallel execution
- Parallelism doesn't create contention
- Source/target systems can handle the workload

---

# 20. Dependency and Data Correctness

Performance is not the only consideration.

Suppose:

```text
Transform
   ↓
Load Gold
```

Load Gold should not start before Transform has produced valid output.

Therefore:

```text
Transform → Load Gold
```

is required for correctness.

Never remove a real dependency merely to improve performance.

### Professional rule

> **Correctness first, then optimize the dependency graph for safe parallelism.**

---

# 21. Dependencies and Validation

A common architecture:

```text
Source A ──┐
           ├──→ Validation ──→ Transformation
Source B ──┘
```

Validation requires both sources.

Therefore:

```text
Source A
   +
Source B
   ↓
Validation
```

If either required source fails:

```text
Validation
```

may not be eligible depending on its configured `Run If`.

Transformation then depends on the validation outcome.

---

# 22. Dependency and Business Criticality

Not every downstream task has the same business importance.

Example:

```text
                 Validation
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
   Financial Gold          Operational Audit
```

Suppose Financial Gold requires strict validation.

If a critical validation rule fails:

```text
Financial Gold → BLOCK
```

But an independent audit task may still be allowed to execute if its own prerequisites are satisfied.

### Professional principle

> **Don't propagate failure farther than the actual business dependency requires.**

---

# 23. Dependency Anti-Patterns

## Anti-pattern 1 — Artificial sequencing

```text
A → B → C → D
```

when tasks are independent.

### Problem

Unnecessary runtime.

### Better

Build the true dependency graph.

---

## Anti-pattern 2 — One global failure gate

```text
Any task fails
      ↓
Everything stops
```

### Problem

Unrelated work is blocked.

### Better

Isolate failure according to actual dependencies.

---

## Anti-pattern 3 — Dependency used as notification logic

Don't create unnecessary dependencies just to send an alert.

For example:

```text
A
↓
B
↓
Alert
```

if the alert only needs to know whether B failed.

Instead model:

```text
B
↓
Alert
```

with the appropriate condition.

---

## Anti-pattern 4 — Dependency used as retry logic

A dependency does not replace retry configuration.

```text
A → B
```

doesn't mean A automatically retries.

Retry behavior is configured separately.

---

# 24. Dependency + Retry

Consider:

```text
A → B
```

A fails because of a temporary network issue.

If A has retries:

```text
A attempt 1 ❌
      ↓
A retry
      ↓
A SUCCESS
      ↓
B becomes eligible
```

The dependency remains:

```text
A → B
```

Retry affects the execution of A; it doesn't change the dependency graph.

---

# 25. Dependency + Repair

Suppose:

```text
A → B → C
```

and:

```text
A = SUCCESS
B = FAILED
C = SKIPPED
```

After fixing the cause of B's failure, targeted recovery can rerun the necessary failed/skipped work according to the Job's repair behavior.

Conceptually:

```text
A ✅
 ↓
B ❌ → repair
 ↓
B ✅
 ↓
C becomes eligible/re-runs as appropriate
```

The goal is to avoid unnecessarily rerunning work that doesn't need to be repeated.

---

# 26. Dependency + For Each

Consider:

```text
Generate Folders
       ↓
For Each
       ↓
500 folders
```

The dependency is:

```text
Generate Folders → For Each
```

Inside For Each:

```text
Folder 1
Folder 2
Folder 3
...
Folder 500
```

can execute according to the configured For Each concurrency.

### Important distinction

The dependency controls:

```text
Generate Folders
        ↓
For Each starts
```

The For Each concurrency controls:

```text
How many iterations execute in parallel?
```

---

# 27. Dependency + Job Concurrency

These are different levels.

### Task dependency

```text
A → B
```

Controls relationships **inside one Job Run**.

### Job concurrency

Controls how many Job Runs may overlap.

Example:

```text
Run #100
   A → B

Run #101
   A → B
```

These are two separate Job Runs with the same task dependency graph.

---

# 28. Professional Decision Framework

When you see a dependency question, ask:

```text
1. Does the downstream task actually need the upstream task?
          ↓
       YES → dependency

2. Can the tasks run independently?
          ↓
       YES → avoid unnecessary dependency

3. What happens if upstream fails?
          ↓
       Check Run If / business requirement

4. Does another branch really depend on it?
          ↓
       If NO → isolate the failure

5. Can branches execute safely in parallel?
          ↓
       YES → exploit parallelism

6. Does parallelism create resource contention?
          ↓
       If YES → control concurrency

7. Is the dependency required for correctness?
          ↓
       NEVER remove it merely for speed
```

---

# 29. Exam Decision Rules

### Question:

> Two tasks don't depend on each other. What should you do?

**Answer:**

Allow them to execute independently rather than creating an unnecessary dependency.

---

### Question:

> Task C requires both A and B to finish successfully.

Architecture:

```text
A ──┐
    ├──→ C
B ──┘
```

Use an appropriate condition such as:

```text
ALL SUCCEEDED
```

when that is the business requirement.

---

### Question:

> A fails but an unrelated branch can still proceed.

Don't block the unrelated branch.

Use dependency isolation.

---

### Question:

> A downstream task should run only when its upstream task fails.

Use:

```text
Dependency → upstream task
Run If → failure condition
```

---

### Question:

> A task should execute after all upstream tasks finish regardless of their result.

Use the appropriate:

```text
Run If = ALL DONE
```

---

### Question:

> You want to control how many repeated inputs execute in parallel.

Use:

```text
For Each concurrency
```

not Job Run concurrency.

---

# 30. Key Terminology

| Term | Meaning |
|---|---|
| Task | Unit of work |
| Upstream | Task this task depends on |
| Downstream | Task that depends on this task |
| Dependency | Execution relationship |
| DAG | Directed Acyclic Graph |
| Parallel branch | Independent execution path |
| Run If | Condition based on upstream outcomes |
| Failure isolation | Prevent unrelated work from being blocked |
| Retry | Re-execution after failure |
| Repair | Targeted recovery |
| For Each | Repeated task execution over inputs |
| Job concurrency | Number of overlapping Job Runs |
| For Each concurrency | Parallel iterations within For Each |

---

# 31. Professional Mental Model

Always think:

```text
                 BUSINESS REQUIREMENT
                         ↓
                  DATA DEPENDENCY?
                    /          \
                  YES           NO
                   ↓             ↓
             CREATE EDGE     KEEP INDEPENDENT
                   ↓             ↓
              RUN IF?       PARALLEL POSSIBLE?
                   ↓             ↓
             FAILURE PATH     YES → PARALLEL
                   ↓             ↓
              RECOVERY        CONTROL CONCURRENCY
```

The goal is:

> **Build the smallest dependency graph that guarantees correctness.**

That gives you:

- Correct execution
- Maximum safe parallelism
- Better SLA
- Lower unnecessary runtime
- Better failure isolation
- Easier recovery

---

# 32. Quick Revision

```text
Dependency
→ "Who depends on whom?"

Upstream
→ "What do I depend on?"

Downstream
→ "Who depends on me?"

Run If
→ "Given the upstream outcome, should I run?"

Parallel branches
→ "Can independent work proceed simultaneously?"

Failure isolation
→ "Does this failure actually affect this branch?"

For Each
→ "Same logic over many inputs"

Job concurrency
→ "How many Job Runs can overlap?"

For Each concurrency
→ "How many iterations can overlap?"

Retry
→ "Can this transient failure recover automatically?"

Repair
→ "How do I recover failed work without unnecessary reprocessing?"
```

---

# 33. Final Exam Rule

> **Do not optimize a DAG by removing a dependency until you have proved that the dependency is not required for data correctness or business correctness.**

The best Professional answer is usually:

```text
Correctness
    ↓
True dependencies
    ↓
Safe parallelism
    ↓
Controlled concurrency
    ↓
Performance optimization
```

---
