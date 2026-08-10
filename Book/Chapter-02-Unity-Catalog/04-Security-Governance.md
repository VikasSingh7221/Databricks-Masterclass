# 04 - Security and Governance

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Unity Catalog Security Model
- Role-Based Access Control (RBAC)
- Users, Groups and Service Principals
- Ownership
- Privileges
- GRANT & REVOKE
- Privilege Inheritance
- Future Grants
- Row Filters
- Column Masks
- Dynamic Views
- Lineage
- Audit Logs
- Enterprise Governance Best Practices

---

# Introduction

Data governance is one of the most important capabilities of Unity Catalog.

A modern organization must answer questions like:

- Who can access this table?
- Who modified this table?
- Who viewed sensitive data?
- Which dashboard uses this table?
- Which downstream systems are affected if this table changes?

Unity Catalog provides centralized governance to answer these questions.

---

# Security Architecture

```
                  Users

                    │

         ┌──────────┴───────────┐

         ▼                      ▼

     Groups              Service Principals

                    │

                    ▼

             Unity Catalog

                    │

      Authentication & Authorization

                    │

                    ▼

      Catalog → Schema → Table → Data
```

---

# Role-Based Access Control (RBAC)

Unity Catalog follows the principle of **Role-Based Access Control (RBAC)**.

Instead of assigning permissions directly to users, permissions are assigned to groups.

Example

```
Users

↓

analyst_group

↓

SELECT

↓

Sales Tables
```

Benefits

- Easier administration
- Consistent permissions
- Better scalability
- Simpler onboarding and offboarding

---

# Users, Groups and Service Principals

## User

Represents an individual person.

Example

```
Alice

Bob

John
```

---

## Group

A collection of users with similar responsibilities.

Examples

```
analyst_group

engineering_group

finance_group
```

Groups simplify permission management.

---

## Service Principal

A non-human identity used by applications or automation.

Examples

- CI/CD pipelines
- Scheduled Jobs
- External Applications

---

# Ownership

Every Unity Catalog object has an owner.

The owner can:

- Manage permissions
- Transfer ownership
- Modify the object
- Delete the object (subject to dependencies)

Ownership is different from privileges.

---

# Privileges

Privileges define **what actions are allowed**.

Common privileges include:

```
USE CATALOG

USE SCHEMA

SELECT

MODIFY

CREATE TABLE

CREATE VIEW

READ FILES

WRITE FILES
```

Different objects support different privileges.

---

# GRANT

Permissions are assigned using GRANT.

Example

```sql
GRANT SELECT
ON TABLE sales.gold.orders
TO `analyst_group`;
```

This allows members of the analyst group to query the table.

---

# REVOKE

Permissions are removed using REVOKE.

Example

```sql
REVOKE SELECT
ON TABLE sales.gold.orders
FROM `analyst_group`;
```

---

# Authorization Flow

Whenever a query is executed:

```
User

↓

Authentication

↓

Unity Catalog

↓

Authorization

↓

Privilege Check

↓

Access Granted?

↓

YES

↓

Read Data
```

If permission is missing, access is denied before Spark reads the data.

---

# Privilege Inheritance

Unity Catalog uses a hierarchical object model.

```
Catalog

↓

Schema

↓

Table
```

To query a table, a user typically requires:

- USE CATALOG
- USE SCHEMA
- SELECT

Access to the Catalog alone is not sufficient.

---

# Future Grants

Organizations frequently create new tables.

Without Future Grants:

```
Create Table

↓

Administrator

↓

GRANT

↓

Repeat
```

With Future Grants:

```
Create Table

↓

Permissions Applied Automatically
```

Future Grants reduce administrative effort and improve consistency.

---

# Principle of Least Privilege

Users should receive only the permissions required for their responsibilities.

Examples

Analyst

```
SELECT
```

Engineer

```
SELECT

MODIFY

CREATE TABLE
```

Platform Administrator

```
Ownership

Permission Management
```

Never grant more permissions than necessary.

---

# Row Filters

A Row Filter limits which rows are returned to a user.

Example

Table

```
employees
```

Rows

| Employee | Department |
|----------|------------|
| Alice | HR |
| Bob | Sales |
| John | Finance |

HR User

```
SELECT *
```

Result

| Employee | Department |
|----------|------------|
| Alice | HR |

Sales and Finance rows are automatically filtered.

---

# Column Masks

A Column Mask hides or transforms sensitive values.

Example

| Name | Salary |
|------|---------|
| Alice | 100000 |

HR

```
100000
```

Intern

```
******
```

The row remains visible.

Only the value changes.

---

# Dynamic Views

Dynamic Views allow query results to change based on the user.

Example

Doctor

```
See Everything
```

Receptionist

```
Mask Diagnosis
```

Regional Manager

```
Only Local Patients
```

One View.

Different results.

---

# Row Filters vs Column Masks vs Dynamic Views

| Feature | Row Filter | Column Mask | Dynamic View |
|----------|------------|-------------|--------------|
| Hide Rows | ✅ | ❌ | ✅ |
| Hide Values | ❌ | ✅ | ✅ |
| User-based Logic | Limited | Limited | Advanced |
| Complex Conditions | ❌ | ❌ | ✅ |

---

# Lineage

Lineage tracks the movement of data across the Lakehouse.

Example

```
CSV

↓

Bronze

↓

Silver

↓

Gold

↓

Dashboard
```

Benefits

- Impact Analysis
- Root Cause Analysis
- Compliance
- Easier Debugging

---

# Audit Logs

Audit Logs record important activities such as:

- User Login
- Table Access
- Permission Changes
- Object Creation
- Object Deletion

Typical information includes:

```
Who

What

When

Which Object
```

Audit Logs support security and compliance requirements.

---

# Enterprise Governance Example

```
Users

↓

Groups

↓

RBAC

↓

Unity Catalog

↓

Row Filters

↓

Column Masks

↓

Dynamic Views

↓

Tables

↓

Audit Logs

↓

Lineage
```

This architecture provides centralized governance across the Lakehouse.

---

# Best Practices

✅ Use Groups instead of individual user permissions.

✅ Follow the Principle of Least Privilege.

✅ Apply Row Filters for row-level security.

✅ Apply Column Masks for sensitive columns.

✅ Use Dynamic Views for complex security rules.

✅ Enable Audit Logs for compliance.

✅ Use Lineage for impact analysis.

---

# Common Mistakes

❌ Granting permissions directly to users.

❌ Giving Administrator privileges to everyone.

❌ Creating duplicate tables instead of using Row Filters.

❌ Hiding rows with Column Masks.

❌ Forgetting to audit sensitive data access.

---

# Associate Exam Notes

Remember:

RBAC

```
Users

↓

Groups

↓

Permissions
```

Security Features

| Requirement | Feature |
|-------------|----------|
| Hide Rows | Row Filter |
| Hide Column Values | Column Mask |
| Complex Security Logic | Dynamic View |
| Track Data Flow | Lineage |
| Track User Activity | Audit Logs |

---

# Professional Interview Questions

## Question 1

Why should permissions be assigned to Groups instead of individual users?

**Answer**

Groups simplify administration, improve consistency, and scale easily as users join or leave the organization.

---

## Question 2

When would you choose a Dynamic View instead of Row Filters and Column Masks?

**Answer**

Use Dynamic Views when security rules depend on more complex business logic or require combining row-level and column-level conditions in a single SQL view.

---

## Question 3

A compliance officer asks:

> Who viewed salary information last week?

Which Unity Catalog feature provides the answer?

**Answer**

Audit Logs.

---

## Question 4

A dashboard shows incorrect revenue.

How do you identify the source?

**Answer**

Use Lineage to trace the dashboard back through Gold, Silver, Bronze, and ultimately to the original source.

---

# Chapter Summary

```
Users

↓

Groups

↓

Privileges

↓

Unity Catalog

↓

Row Filters

↓

Column Masks

↓

Dynamic Views

↓

Tables

↓

Audit Logs

↓

Lineage
```

---

# Key Takeaways

- Unity Catalog centralizes governance across the Lakehouse.
- RBAC simplifies permission management through Groups.
- Privileges control actions on securable objects.
- Row Filters restrict rows.
- Column Masks hide sensitive values.
- Dynamic Views implement complex user-specific logic.
- Lineage tracks data flow.
- Audit Logs track user activity and permission changes.
- Following the Principle of Least Privilege improves security and reduces risk.