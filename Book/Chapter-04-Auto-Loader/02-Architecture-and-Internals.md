# 02 - Auto Loader Architecture and Internals

> **Difficulty:** ⭐⭐⭐⭐⭐
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this chapter, you will understand:

- Auto Loader Architecture
- Directory Listing Mode
- File Notification Mode
- Internal File Discovery
- Metadata Tracking
- Exactly-Once Processing
- Idempotency
- Commit Protocol
- cloudFiles.allowOverwrites
- Failure Scenarios
- Enterprise Architecture

---

# Auto Loader Internal Architecture

Auto Loader is not simply a command that reads files.

It is an intelligent ingestion system built on top of Spark Structured Streaming.

```
              Cloud Storage

                    │

                    ▼

           File Discovery

                    │

                    ▼

          Structured Streaming

                    │

                    ▼

            Read New Files

                    │

                    ▼

           Write Delta Table

                    │

                    ▼

         Successful Commit ?

            │          │

          YES          NO

           │            │

           ▼            ▼

 Update Metadata    Retry Next Run
```

The most important point is:

**Metadata is updated only after a successful commit.**

---

# Auto Loader Components

```
Auto Loader

│

├── File Discovery

├── Metadata Tracker

├── Schema Manager

├── Structured Streaming

├── Delta Writer

└── Checkpoint Manager
```

Each component has a dedicated responsibility.

---

# File Discovery

The first responsibility of Auto Loader is discovering new files.

Auto Loader supports two discovery methods.

```
Auto Loader

│

├── Directory Listing

└── File Notification
```

Both eventually provide the same output:

```
List of New Files
```

The difference is **how those files are discovered**.

---

# Directory Listing Mode

Directory Listing is the simplest discovery mechanism.

```
Cloud Storage

↓

List Files

↓

Compare Metadata

↓

Identify New Files

↓

Read New Files
```

Auto Loader lists the files in the configured directory and compares them with previously processed metadata.

Files already processed are skipped.

Only new files are read.

---

## Example

Suppose the storage contains

```
001.csv

002.csv

003.csv
```

First run

```
Load

001

002

003
```

Metadata

```
001

002

003
```

Now

```
004.csv
```

arrives.

Next execution

Auto Loader compares

```
001

002

003

004
```

against metadata

```
001

002

003
```

Only

```
004
```

is processed.

---

# File Notification Mode

Directory Listing becomes slower when billions of files exist.

Instead of repeatedly listing storage,

cloud services can notify Auto Loader whenever a new file arrives.

Architecture

```
Cloud Storage

↓

Cloud Notification

↓

Auto Loader

↓

Read File
```

Examples

AWS

```
Amazon S3

↓

SQS

↓

Auto Loader
```

Azure

```
ADLS

↓

Event Grid

↓

Queue

↓

Auto Loader
```

Google Cloud

```
GCS

↓

Pub/Sub

↓

Auto Loader
```

This avoids repeatedly listing the storage location.

---

# Which Mode Should You Use?

| Directory Listing | File Notification |
|-------------------|------------------|
| Easy to configure | More configuration |
| Good for smaller workloads | Best for very large workloads |
| Periodic listing | Event-driven |
| Higher listing cost | Lower listing cost |

---

# Internal Processing Flow

Suppose

```
005.csv
```

arrives.

```
File Arrives

↓

Auto Loader Detects File

↓

Read File

↓

Validate Schema

↓

Write Bronze

↓

Commit

↓

Update Metadata
```

Every successful file follows this lifecycle.

---

# Metadata Tracking

One of the biggest strengths of Auto Loader is metadata tracking.

Auto Loader stores information about processed files.

Examples

```
001.csv

002.csv

003.csv
```

When the next execution starts,

Auto Loader checks

```
Already Processed?

↓

YES

↓

Skip

---------------

NO

↓

Process
```

This avoids duplicate ingestion.

---

# Idempotency

Idempotency means that executing the same operation multiple times produces the same final result.

Example

```
001.csv
```

already processed.

If Auto Loader sees it again,

```
Metadata Exists

↓

Skip File
```

The table remains unchanged.

No duplicate data is written.

---

# Exactly-Once Processing

Auto Loader guarantees file-level exactly-once processing.

The sequence is critical.

```
Read File

↓

Write Delta

↓

Commit Successful

↓

Update Metadata
```

Metadata is **never updated before the commit**.

---

# Why?

Suppose metadata were updated first.

```
Read File

↓

Update Metadata

↓

Cluster Crash
```

Next execution

```
Metadata Exists

↓

Skip File
```

But the data was never written.

Result

```
Data Loss
```

---

Instead,

Auto Loader performs

```
Read

↓

Write

↓

Commit

↓

Metadata Update
```

If a crash occurs before commit,

metadata is not updated.

Next execution

```
File

↓

Processed Again
```

No data is lost.

---

# Failure Scenario

Suppose

```
005.csv
```

is being processed.

```
Read File

↓

Write Started

↓

Cluster Crash
```

Metadata has **not** been updated.

Next execution

```
005.csv

↓

Processed Again
```

Result

No missing data.

This is fault tolerance.

---

# cloudFiles.allowOverwrites

By default

```
allowOverwrites = false
```

Suppose

```
orders.csv
```

has already been processed.

Someone uploads another file with the same name.

Auto Loader checks metadata.

```
orders.csv

↓

Already Exists

↓

Skip
```

---

If

```
allowOverwrites = true
```

Auto Loader allows the same filename to be processed again.

```
orders.csv

↓

Process Again
```

This is useful when upstream systems overwrite files.

However,

it can introduce duplicate records if not handled carefully.

---

# Enterprise Example

Suppose a banking system receives transaction files every minute.

```
Transactions

↓

Amazon S3

↓

Auto Loader

↓

Bronze

↓

Silver

↓

Gold
```

Every transaction file is processed exactly once.

Even if the cluster crashes,

processing resumes safely from the checkpoint.

---

# Best Practices

✅ Store checkpoints in durable cloud storage.

✅ Keep Bronze append-only.

✅ Use File Notification for very large workloads.

✅ Monitor schema evolution.

✅ Keep ingestion logic lightweight.

---

# Common Mistakes

❌ Deleting checkpoint directories.

❌ Running business transformations during ingestion.

❌ Setting allowOverwrites=true without understanding duplicate risks.

❌ Assuming metadata is updated before writing.

---

# Associate Exam Notes

Remember

```
Directory Listing

↓

Lists Files

----------------

File Notification

↓

Receives Events

----------------

Metadata

↓

Tracks Processed Files

----------------

Commit

↓

Metadata Updated AFTER Success
```

---

# Professional Interview Questions

## Question 1

Why does Auto Loader update metadata only after a successful commit?

### Expected Answer

Updating metadata before a successful commit could cause data loss if the job fails after recording the file as processed but before writing it to the destination. Updating metadata only after a successful commit guarantees fault tolerance and exactly-once processing.

---

## Question 2

When would you use File Notification Mode instead of Directory Listing?

### Expected Answer

File Notification Mode is preferred for very large-scale environments because it avoids repeated directory scans and reduces cloud storage API costs by using cloud-native event notifications.

---

## Question 3

What happens if the cluster crashes after writing starts but before metadata is updated?

### Expected Answer

Since metadata has not yet been updated, Auto Loader treats the file as unprocessed during the next execution and safely retries processing, preventing data loss.

---

# Chapter Summary

```
Cloud Storage

↓

File Discovery

↓

Read File

↓

Write Delta

↓

Commit

↓

Update Metadata
```

---

# Key Takeaways

- Auto Loader supports Directory Listing and File Notification modes.
- Metadata tracks processed files.
- Metadata is updated only after a successful commit.
- This guarantees exactly-once file ingestion.
- Idempotency prevents duplicate processing of previously ingested files.
- `cloudFiles.allowOverwrites` allows reprocessing of files with the same name but must be used carefully.