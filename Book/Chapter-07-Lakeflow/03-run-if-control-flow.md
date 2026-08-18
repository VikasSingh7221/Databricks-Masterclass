# 03. Run If & Control Flow

> **Databricks Data Engineer Professional — Certification Notes**

---

# 1. What is `Run if`?

`Run if` controls whether a task should execute based on the outcome of its upstream dependencies.

It answers:

> **"Given what happened upstream, should this task run?"**

It does **not** define the dependency itself.

Example:

```text
A ──→ B
```

The dependency says:

> B depends on A.

Then:

```text
B
Run if = All succeeded
```

says:

> B should execute only when the required upstream condition is satisfied.

---

# 2. Dependency vs Run If

This distinction is extremely important.

## Dependency

```text
A ──→ B
```

Answers:

> **Which task does B depend on?**

## Run If

```text
B
Run if = At least one failed
```

Answers:

> **Given the outcomes of B's upstream tasks, should B execute?**

### Mental model

```text
Dependency
    ↓
WHO?

Run If
    ↓
UNDER WHAT OUTCOME?
```

---

# 3. Available Run If Conditions

Databricks provides several `Run if` conditions for task dependencies.

| Condition | Meaning |
|---|---|
| **All succeeded** | All dependencies succeeded |
| **At least one succeeded** | At least one dependency succeeded |
| **None failed** | No dependency failed and at least one dependency ran |
| **All done** | All dependencies completed, regardless of outcome |
| **At least one failed** | At least one dependency failed |
| **All failed** | All dependencies failed |

> Always verify the current Databricks documentation for exact behavior because workflow semantics can evolve.

Official documentation:  
https://docs.databricks.com/aws/en/jobs/run-if

---

# 4. All Succeeded

Use:

```text
Run if = ALL SUCCEEDED
```

when the task requires **every upstream dependency to succeed**.

Example:

```text
Salesforce ──┐
             ├──→ Validation
Snowflake ───┘
```

If Validation requires both sources:

```text
Validation
Run if = All succeeded
```

### Outcomes

```text
Salesforce = SUCCESS
Snowflake  = SUCCESS
        ↓
Validation → RUN
```

But:

```text
Salesforce = SUCCESS
Snowflake  = FAILED
        ↓
Validation → NOT RUN
```

---

# 5. At Least One Succeeded

Use:

```text
Run if = AT LEAST ONE SUCCEEDED
```

when the downstream task can proceed as long as at least one upstream dependency succeeds.

Example:

```text
Source A ──┐
           ├──→ Fallback Processing
Source B ──┘
```

If the business logic allows processing from either source:

```text
A = SUCCESS
B = FAILED
```

then:

```text
Fallback Processing → RUN
```

### Important

Do not choose this simply because:

> "One source succeeded."

The business requirement must actually allow partial upstream success.

---

# 6. None Failed

Use:

```text
Run if = NONE FAILED
```

when the task should run if there are no failed upstream tasks and at least one upstream task has run.

This is different from:

```text
ALL SUCCEEDED
```

because the semantics around upstream outcomes such as exclusions/cancellations matter.

### Professional rule

When a question gives multiple possible upstream states, carefully inspect:

```text
SUCCESS
FAILED
EXCLUDED
CANCELED
```

Do not assume all non-success states behave identically.

---

# 7. All Done

Use:

```text
Run if = ALL DONE
```

when the downstream task should execute after all upstream tasks have completed, regardless of whether they succeeded or failed.

Example:

```text
Task A ──┐
         ├──→ Cleanup
Task B ──┘
```

Requirement:

> Cleanup must happen after A and B finish, regardless of their outcomes.

Then:

```text
Cleanup
Run if = ALL DONE
```

### Mental model

```text
A → SUCCESS
B → FAILED

Both finished
     ↓
Cleanup → RUN
```

This is useful for:

- Cleanup
- Final reporting
- Audit processing
- Certain operational tasks

---

# 8. At Least One Failed

Use:

```text
Run if = AT LEAST ONE FAILED
```

when the task should execute if any upstream dependency fails.

Example:

```text
Task A ──┐
         ├──→ Failure Handler
Task B ──┘
```

If:

```text
A = SUCCESS
B = FAILED
```

then:

```text
Failure Handler → RUN
```

---

# 9. All Failed

Use:

```text
Run if = ALL FAILED
```

when the downstream task should execute only when every upstream dependency failed.

Example:

```text
Source A ──┐
           ├──→ Escalation
Source B ──┘
```

If:

```text
A = FAILED
B = FAILED
```

then:

```text
Escalation → RUN
```

But:

```text
A = FAILED
B = SUCCESS
```

means:

```text
Escalation → NOT RUN
```

because not all upstream tasks failed.

---

# 10. Run If Decision Table

For two upstream tasks:

```text
A
B
```

consider:

| A | B | All Succeeded | At Least One Succeeded | At Least One Failed | All Failed |
|---|---|---|---|---|---|
| SUCCESS | SUCCESS | RUN | RUN | NO | NO |
| SUCCESS | FAILED | NO | RUN | RUN | NO |
| FAILED | SUCCESS | NO | RUN | RUN | NO |
| FAILED | FAILED | NO | NO | RUN | RUN |

This table is extremely useful for exam questions.

---

# 11. All Done Example

```text
A ──┐
    ├──→ D
B ──┘
```

Suppose:

```text
A = SUCCESS
B = FAILED
```

If:

```text
D = ALL DONE
```

then D can run because:

```text
A → finished
B → finished
```

The success/failure result isn't the requirement.

The requirement is:

> **Both upstream tasks must be finished.**

---

# 12. Failure Handler Pattern

A common architecture:

```text
             ┌──→ Transformation
Validation ──┤
             └──→ Failure Handler
```

Transformation:

```text
Run if = SUCCESS
```

Failure Handler:

```text
Run if = FAILED
```

Conceptually:

```text
Validation
    │
    ├── SUCCESS → Transformation
    │
    └── FAILED  → Failure Handler
```

This creates explicit success/failure branches.

---

# 13. Success + Failure Branching

Example:

```text
                 Validation
                    │
             ┌──────┴──────┐
             ↓             ↓
       Transformation   Alert
          SUCCESS        FAILED
```

Configuration:

```text
Transformation
→ Run if = All succeeded

Alert
→ Run if = At least one failed
```

This lets the workflow react differently depending on the validation result.

---

# 14. Important: A Failure Handler Does Not Automatically Fix the Failure

Suppose:

```text
Validation → FAILED
        ↓
Failure Handler
```

The Failure Handler running does not mean Validation succeeded.

It simply means:

> The workflow intentionally executed a task because the failure condition was met.

This distinction is important.

```text
Failure
   ↓
Failure handler runs
   ≠
Failure is resolved
```

---

# 15. Run If and Skipped/Excluded Tasks

This is a Professional-level area where you must be precise.

A task whose `Run if` condition is not satisfied can become **Excluded** rather than executing.

For example:

```text
A → B
```

If B has:

```text
Run if = At least one failed
```

and:

```text
A = SUCCESS
```

then B's condition is not satisfied.

Therefore B is excluded from execution.

### Important terminology

Current Databricks documentation uses **Excluded** for a task that is not run because its `Run if` condition is not satisfied.

Do not automatically treat:

```text
FAILED
```

and:

```text
EXCLUDED
```

as the same state.

---

# 16. Excluded Upstream Tasks

This is an important exam trap.

Suppose:

```text
A → B → C
```

and B becomes excluded.

Then C's `Run if` evaluation depends on the documented handling of excluded upstream tasks.

Current Databricks documentation specifies that an upstream task with the `Excluded` outcome is treated as **successful** when evaluating `Run if` conditions.

By contrast, upstream failures and cancellations are treated as failures.

Therefore, do not reason:

> "B didn't run, so B automatically counts as failed."

That is incorrect.

---

# 17. Why This Matters

Consider:

```text
A
↓
B
↓
C
```

Suppose:

```text
B = EXCLUDED
```

and C uses:

```text
Run if = ALL SUCCEEDED
```

You need to apply Databricks' documented semantics for the `Excluded` state rather than assuming:

```text
Excluded = Failed
```

This is exactly the type of state-handling detail that can appear in Professional-level scenario questions.

---

# 18. Run If and Cancellation

Cancellation is another state that should not automatically be treated as success.

For Professional questions, distinguish:

```text
SUCCESS
FAILED
EXCLUDED
CANCELED
```

When the scenario involves cancellation, refer to the documented `Run if` semantics rather than relying on intuition.

### Exam rule

> **Never collapse all non-success outcomes into "failed."**

---

# 19. Control Flow: If/Else

Lakeflow Jobs can use conditional logic to create branches based on an expression.

Conceptually:

```text
                Condition
                    ↓
              ┌─────┴─────┐
             TRUE        FALSE
              ↓            ↓
           Task A        Task B
```

Example:

```text
record_count > 0
```

If true:

```text
Process Data
```

If false:

```text
No Data Handler
```

---

# 20. If/Else vs Run If

These are different mechanisms.

## Run If

Uses **upstream task outcomes**.

Example:

```text
Validation
   ↓
Alert

Run if = FAILED
```

It answers:

> Did the upstream task succeed/fail/etc.?

---

## If/Else

Evaluates a **condition/expression**.

Example:

```text
record_count > 0
```

It answers:

> Is this expression true or false?

### Mental model

```text
Run If
→ workflow state

If/Else
→ logical/data condition
```

---

# 21. Example: Validation Branch

Suppose:

```text
Validation
   ↓
record_count
```

You want:

```text
record_count > 0
```

If:

```text
TRUE
```

then:

```text
Process Data
```

If:

```text
FALSE
```

then:

```text
No Data Alert
```

This is better represented as conditional logic rather than pretending the validation task itself failed.

---

# 22. Technical Failure vs Business Condition

This distinction is extremely important.

Suppose:

```text
Query executes successfully
```

but returns:

```text
record_count = 0
```

Technically:

```text
Task status = SUCCESS
```

But business logic may say:

```text
No records
   ↓
Warning
```

Therefore:

```text
Technical status
        ≠
Business outcome
```

A task can succeed technically while the business condition indicates something needs attention.

---

# 23. Example: Critical Validation

Suppose:

```text
Overall completeness = 99.8%
```

and the general threshold is:

```text
≥ 99%
```

So technically:

```text
PASS
```

But:

```text
Critical financial records = 99.2%
Required = 100%
```

Therefore:

```text
Critical validation = FAIL
```

The downstream financial transformation should be blocked if it depends on that validation.

However, unrelated/audit work may still proceed if its dependencies are satisfied.

---

# 24. Branching by Business Criticality

A sophisticated architecture may look like:

```text
                    Validation
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
       Critical Data          Non-critical Data
              │                   │
           Validate             Validate
              │                   │
        PASS / FAIL           PASS / WARN
              │                   │
              ↓                   ↓
       Financial Gold        Operational Gold
```

This prevents a minor issue from unnecessarily blocking unrelated business processes.

---

# 25. Run If + Multiple Upstream Tasks

Suppose:

```text
A ──┐
    ├──→ C
B ──┘
```

C has:

```text
Run if = AT LEAST ONE SUCCEEDED
```

Possible outcomes:

```text
A = SUCCESS
B = FAILED
```

C can run.

But this is only correct if the business logic permits partial upstream success.

### Professional trap

Never select:

```text
AT LEAST ONE SUCCEEDED
```

merely because it allows more execution.

The requirement must justify it.

---

# 26. Failure Alert Pattern

Suppose:

```text
Validation
    ↓
Alert
```

and:

```text
Alert
Run if = FAILED
```

Then:

```text
Validation = FAILED
       ↓
Alert executes
```

But:

```text
Validation = SUCCESS
       ↓
Alert does not execute
```

This is a clean, targeted failure notification pattern.

---

# 27. Cleanup Pattern

Suppose:

```text
Extract ──┐
          ├──→ Cleanup
Transform ┘
```

Requirement:

> Cleanup should execute after both tasks finish, regardless of success/failure.

Use:

```text
Cleanup
Run if = ALL DONE
```

Conceptually:

```text
Extract     → SUCCESS
Transform   → FAILED
                ↓
             Cleanup
                ↓
               RUN
```

This is a classic use case for `ALL DONE`.

---

# 28. Failure Escalation Pattern

Suppose:

```text
Source A ──┐
           ├──→ Escalation
Source B ──┘
```

Requirement:

> Escalate only if both sources fail.

Use:

```text
Escalation
Run if = ALL FAILED
```

If:

```text
A = FAILED
B = FAILED
```

then:

```text
Escalation → RUN
```

But if:

```text
A = FAILED
B = SUCCESS
```

then:

```text
Escalation → NOT RUN
```

---

# 29. "At Least One Failed" vs "All Failed"

This is a common exam trap.

### At least one failed

```text
A = FAILED
B = SUCCESS
```

→ **RUN**

because one failed.

### All failed

```text
A = FAILED
B = SUCCESS
```

→ **DO NOT RUN**

because B succeeded.

Only:

```text
A = FAILED
B = FAILED
```

causes `ALL FAILED` to run.

---

# 30. "All Succeeded" vs "None Failed"

These should not be treated as interchangeable.

### All succeeded

Requires all upstream tasks to satisfy the success condition.

### None failed

Focuses on whether any upstream task failed, while the documented semantics also require at least one upstream task to have run.

For certification questions involving:

- Excluded tasks
- Canceled tasks
- Mixed states

apply the current Databricks semantics explicitly.

---

# 31. Control Flow Architecture

A production workflow can combine several control-flow mechanisms:

```text
                    Ingestion
                        ↓
                   Validation
                        ↓
                  ┌─────┴─────┐
                  ↓           ↓
              SUCCESS       FAILED
                  ↓           ↓
             If/Else       Alert
                  ↓
          ┌───────┴───────┐
          ↓               ↓
       Data > 0        Data = 0
          ↓               ↓
      Transform        No Data
```

This combines:

```text
Dependencies
+
Run If
+
If/Else
```

---

# 32. Control Flow Decision Framework

When you see a scenario, ask:

```text
1. Is the decision based on upstream TASK STATUS?
          ↓
       Run If

2. Is the decision based on an EXPRESSION?
          ↓
       If/Else

3. Does the downstream task require ALL upstream tasks?
          ↓
       All Succeeded

4. Can it run if ANY upstream task succeeds?
          ↓
       At Least One Succeeded

5. Should it run after everything finishes?
          ↓
       All Done

6. Should it run when something fails?
          ↓
       At Least One Failed

7. Should it run only when everything fails?
          ↓
       All Failed
```

---

# 33. Professional Exam Rules

### Rule 1

> **Dependency defines the relationship; Run If defines the allowed upstream outcome.**

---

### Rule 2

> **Use All Succeeded when every required upstream task must succeed.**

---

### Rule 3

> **Use At Least One Failed for a failure-handler branch when any upstream failure is enough to trigger it.**

---

### Rule 4

> **Use All Done for cleanup/finalization that must happen after upstream completion regardless of result.**

---

### Rule 5

> **Do not treat Excluded as Failed.**

---

### Rule 6

> **Don't confuse technical task success with business success.**

---

### Rule 7

> **Use If/Else for expression-based branching; use Run If for upstream task outcomes.**

---

### Rule 8

> **Do not block unrelated branches simply because another branch failed.**

---

# 34. Common Exam Traps

## Trap 1

Question:

> "Run when B fails."

Wrong:

```text
Run If = All Done
```

Correct:

```text
Run If = At Least One Failed
```

if B is the relevant upstream dependency.

---

## Trap 2

Question:

> "Run after A and B finish, regardless of outcome."

Wrong:

```text
All Succeeded
```

Correct:

```text
All Done
```

---

## Trap 3

Question:

> "Run if at least one source succeeded."

Correct:

```text
At Least One Succeeded
```

but only when partial success is acceptable.

---

## Trap 4

Question:

> "Run only when every source failed."

Correct:

```text
All Failed
```

---

## Trap 5

Question:

> "Run when record_count > 0."

This is an expression-based condition.

Use:

```text
If/Else
```

rather than treating the record count as a task status.

---

## Trap 6

Question:

> "Task was skipped/excluded, therefore it failed."

Incorrect.

Current Databricks semantics distinguish **Excluded** from **Failed**.

---

# 35. Quick Reference Table

| Requirement | Appropriate mechanism |
|---|---|
| Task depends on another task | Dependency |
| Run only if all upstream succeed | All Succeeded |
| Run if any upstream succeeds | At Least One Succeeded |
| Run if no upstream failed | None Failed |
| Run after all upstream complete | All Done |
| Run if any upstream fails | At Least One Failed |
| Run only if all upstream fail | All Failed |
| Branch based on expression | If/Else |
| Process many inputs | For Each |
| Handle transient failure | Retry |
| Recover failed work | Repair |
| Notify on failure | Notification / failure branch |

---

# 36. Final Mental Model

Remember this:

```text
                    JOB
                     │
                  Trigger
                     │
                  Job Run
                     │
                   Tasks
                     │
              ┌──────┴──────┐
              ↓             ↓
        Dependencies     Independent
              │             │
              ↓             ↓
           Run If        Parallel work
              │
              ↓
          If / Else
              │
              ↓
          Branching
```

### The core distinction

```text
TRIGGER
→ When does the Job start?

DEPENDENCY
→ Which tasks are related?

RUN IF
→ Should this task run based on upstream outcomes?

IF/ELSE
→ Which branch should execute based on an expression?

FOR EACH
→ Repeat the same logic for many inputs.
```

---

# 37. Professional Scenario Template

For any control-flow question, write:

```text
Upstream tasks:
A, B, C

Business requirement:
What must happen?

Dependency:
Which tasks does downstream work depend on?

Condition:
What upstream state should allow execution?

Run If:
Which condition matches?

Failure behavior:
What happens if upstream fails?

Business impact:
Which downstream branches actually need to stop?

Recovery:
Can the failed branch be repaired independently?
```

This prevents you from choosing an answer merely because it "sounds safe."

---

# 38. Chapter Summary

Lakeflow Jobs control flow is built around:

```text
Dependencies
      +
Run If
      +
If/Else
      +
For Each
```

Use each for its intended purpose.

```text
Dependency
→ relationship

Run If
→ upstream outcome

If/Else
→ expression

For Each
→ repeated processing
```

The Professional goal is not:

> **"Stop everything when anything fails."**

The goal is:

> **"Allow every safe and valid branch to continue while preventing invalid downstream processing."**

---
