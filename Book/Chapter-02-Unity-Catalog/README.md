# Chapter 2 - Unity Catalog

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Unity Catalog Architecture
- Catalogs, Schemas and Tables
- Managed vs External Tables
- Volumes
- Storage Credentials
- External Locations
- RBAC
- Privileges
- Row Filters
- Column Masks
- Dynamic Views
- Lineage
- Audit Logs
- Enterprise Governance

---

# What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for managing data, AI assets, and access control across an organization.

It provides a single place to manage:

- Data Security
- Permissions
- Metadata
- Data Discovery
- Lineage
- Auditing
- Fine-Grained Access Control

Instead of every cluster managing permissions independently, Unity Catalog provides centralized governance.

---

# Why was Unity Catalog Introduced?

Before Unity Catalog, each Databricks workspace managed data independently.

Example:

```
Workspace A

↓

Own Permissions

----------------

Workspace B

↓

Own Permissions
```

Problems:

- Duplicate permission management
- No centralized governance
- Difficult auditing
- Inconsistent security
- Difficult collaboration

Unity Catalog solves these by providing centralized governance.

---

# Unity Catalog Architecture

```
Users / Groups

        │

        ▼

 Unity Catalog

        │

 ┌──────┼─────────┐

 ▼      ▼         ▼

Catalog Schema Storage

        │

        ▼

Delta Tables / Volumes

        │

        ▼

Amazon S3 / ADLS / GCS
```

---

# Unity Catalog Object Hierarchy

```
Metastore

↓

Catalog

↓

Schema

↓

Table

or

Volume
```

Every object belongs to a parent object.

---

# Chapter Contents

This chapter is divided into multiple sections.

| File | Topics |
|------|--------|
| 01-Catalog-Schema.md | Catalogs, Schemas |
| 02-Tables-and-Volumes.md | Managed Tables, External Tables, Volumes |
| 03-Storage.md | Storage Credentials, External Locations |
| 04-Security.md | RBAC, GRANT, REVOKE, Ownership |
| 05-Governance.md | Row Filters, Column Masks, Dynamic Views, Lineage, Audit Logs |
| 06-Exam-Notes.md | Associate Revision |
| 07-Interview-Questions.md | Professional Scenarios |

---

# Key Takeaways

- Unity Catalog centralizes governance.
- It separates governance from compute.
- It manages permissions, metadata, lineage, and auditing.
- It provides fine-grained access control across the Lakehouse.

---
