# 03 - Storage Credentials & External Locations

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Why Storage Credentials are required
- What problem they solve
- What is an External Location?
- Authentication Flow
- Internal Architecture
- AWS IAM Integration
- Enterprise Design
- Best Practices
- Associate Exam Notes
- Professional Interview Questions

---

# Introduction

Databricks stores data in cloud object storage such as:

- Amazon S3
- Azure Data Lake Storage (ADLS)
- Google Cloud Storage (GCS)

Before Unity Catalog, every cluster required direct cloud credentials to access storage.

Problems:

- Duplicate credential management
- Difficult credential rotation
- Poor governance
- Security risks
- Difficult auditing

Unity Catalog solves these problems by introducing **Storage Credentials** and **External Locations**.

---

# Why Were Storage Credentials Introduced?

Suppose ten Databricks clusters need access to the same S3 bucket.

Without Unity Catalog:

```
Cluster A

↓

IAM Role

----------------

Cluster B

↓

IAM Role

----------------

Cluster C

↓

IAM Role
```

Each cluster manages authentication independently.

Problems:

- Multiple IAM configurations
- Higher security risk
- Hard to maintain
- Difficult to audit

Instead, Databricks centralizes authentication.

---

# What is a Storage Credential?

A Storage Credential is a Unity Catalog object that securely stores the cloud authentication mechanism required to access external storage.

It represents **who Databricks is** when accessing cloud storage.

It can use:

- AWS IAM Role
- Azure Managed Identity
- Google Cloud Service Account

---

# Think of it Like This

Storage Credential answers one question:

> **Who am I?**

Example:

```
Databricks

↓

Storage Credential

↓

AWS IAM Role

↓

Amazon S3
```

It contains authentication information only.

It does **not** store data.

---

# Storage Credential Architecture

```
User

↓

Unity Catalog

↓

Storage Credential

↓

Cloud Identity

↓

Amazon S3
```

---

# What is an External Location?

Authentication alone is not enough.

Databricks also needs to know **where** data is stored.

An External Location combines:

- Storage Credential
- Cloud Storage Path

into a reusable object.

---

# Think of it Like This

Storage Credential answers:

```
WHO?
```

External Location answers:

```
WHERE?
```

---

# External Location Architecture

```
Storage Credential

+

s3://company-data/

↓

External Location
```

Example:

```
Storage Credential

↓

CompanyRole

----------------

Storage Path

↓

s3://company-data/bronze/

----------------

External Location

↓

bronze_location
```

---

# Authentication Flow

Suppose a user queries an External Table.

```
User

↓

Unity Catalog

↓

External Location

↓

Storage Credential

↓

AWS IAM Role

↓

Amazon S3

↓

Read Files
```

Notice that the user never directly authenticates with AWS.

Unity Catalog performs the authentication.

---

# Internal Architecture

```
Notebook

↓

SQL Query

↓

Unity Catalog

↓

Permission Check

↓

External Location

↓

Storage Credential

↓

AWS IAM Role

↓

Amazon S3

↓

Return Data
```

---

# Why Separate Storage Credentials and External Locations?

Suppose one IAM Role has access to three buckets.

```
Sales

Finance

Marketing
```

Should you create three Storage Credentials?

No.

One Storage Credential can be reused by multiple External Locations.

Example:

```
Storage Credential

↓

CompanyRole

↓

├── SalesLocation
├── FinanceLocation
└── MarketingLocation
```

Benefits:

- Less duplication
- Easier maintenance
- Centralized authentication

---

# Enterprise Example

Company Storage

```
Amazon S3

↓

company-data

├── bronze
├── silver
└── gold
```

Unity Catalog

```
Storage Credential

↓

CompanyRole

↓

External Locations

↓

bronze_location

silver_location

gold_location
```

One authentication object.

Multiple storage paths.

---

# Managed Tables vs External Tables

Managed Tables

```
Databricks

↓

Manages Storage
```

No External Location required.

---

External Tables

```
Amazon S3

↓

External Location

↓

Storage Credential
```

Authentication is required because the data is outside managed storage.

---

# Best Practices

✅ Reuse Storage Credentials whenever possible.

---

✅ Create separate External Locations for different storage paths.

---

✅ Follow the Principle of Least Privilege for IAM Roles.

---

✅ Avoid embedding cloud credentials directly in notebooks or clusters.

---

# Common Mistakes

❌ Storage Credential stores data.

False.

It stores authentication information.

---

❌ External Location stores IAM credentials.

False.

It references a Storage Credential.

---

❌ One Storage Credential is required for every bucket.

False.

One Storage Credential can be reused by multiple External Locations if appropriate.

---

# Associate Exam Notes

Remember:

```
Storage Credential

↓

WHO?

(Authentication)

-------------------------

External Location

↓

WHERE?

(Storage Path)
```

Storage Credentials manage authentication.

External Locations define accessible cloud paths.

---

# Professional Interview Questions

## Question 1

Why are Storage Credentials and External Locations separate objects?

### Expected Answer

Authentication and storage location are different responsibilities. A single Storage Credential can securely represent cloud authentication, while multiple External Locations reuse that credential for different storage paths. This improves security, reduces duplication, and simplifies administration.

---

## Question 2

A company has one IAM Role with access to three S3 buckets.

Should you create one Storage Credential or three?

### Expected Answer

One Storage Credential is sufficient because it represents the authentication mechanism. Multiple External Locations can reference the same Storage Credential while pointing to different storage paths.

---

## Question 3

Why shouldn't Databricks clusters directly store AWS credentials?

### Expected Answer

Centralizing authentication through Unity Catalog improves security, simplifies credential rotation, enables centralized governance, and reduces operational complexity.

---

# Chapter Summary

```
User

↓

Unity Catalog

↓

External Location

↓

Storage Credential

↓

AWS IAM Role

↓

Amazon S3
```

---

# Key Takeaways

- Storage Credentials define **who** Databricks is when accessing cloud storage.
- External Locations define **where** data is stored.
- Storage Credentials contain authentication, not data.
- External Locations combine a storage path with a Storage Credential.
- One Storage Credential can support multiple External Locations.
- Unity Catalog centralizes secure access to cloud storage.