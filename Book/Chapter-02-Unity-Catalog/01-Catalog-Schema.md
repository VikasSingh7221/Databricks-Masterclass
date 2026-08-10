# 01 - Catalog and Schema

> **Difficulty:** ⭐⭐⭐⭐☆
>
> **Certification:** Databricks Data Engineer Associate & Professional

---

# Learning Objectives

After completing this section, you will understand:

- What is a Catalog?
- What is a Schema?
- Why are Catalogs and Schemas required?
- Unity Catalog Hierarchy
- Enterprise Design
- Best Practices
- Associate Exam Notes
- Professional Interview Questions

---

# Why Do We Need Catalogs and Schemas?

Imagine a company stores all its tables in one location.

```
customers

orders

employees

products

sales

finance

hr

inventory

payments

...
```

As the company grows to thousands of tables, finding and managing data becomes difficult.

Problems:

- No logical organization
- Difficult permission management
- Naming conflicts
- Poor discoverability
- Hard to collaborate

Unity Catalog solves this by organizing data into a hierarchy.

---

# Unity Catalog Hierarchy

```
Metastore

↓

Catalog

↓

Schema

↓

Table
```

Think of it like folders on your computer.

```
Drive

↓

Folder

↓

Subfolder

↓

Files
```

Similarly,

```
Metastore

↓

Catalog

↓

Schema

↓

Tables
```

---

# What is a Catalog?

A Catalog is the highest logical container inside a Metastore.

It is used to organize data based on business domains, departments, projects, or environments.

Examples:

```
sales

finance

hr

marketing

development

production
```

A Catalog can contain multiple Schemas.

---

# Why Use Catalogs?

Without Catalogs:

```
10,000 Tables

↓

One Place
```

With Catalogs:

```
Finance Catalog

Sales Catalog

HR Catalog

Marketing Catalog
```

Benefits:

- Better organization
- Easier security management
- Department isolation
- Easier data discovery

---

# Enterprise Example

```
Company

↓

Metastore

├── finance
├── sales
├── hr
└── marketing
```

Each department owns its own Catalog.

---

# What is a Schema?

A Schema is a logical container inside a Catalog.

It groups related database objects.

A Schema contains:

- Tables
- Views
- Functions
- Volumes (depending on object type and use case)

Example:

```
Sales Catalog

↓

bronze

silver

gold
```

or

```
Finance Catalog

↓

payroll

budget

audit
```

---

# Why Use Schemas?

Suppose Sales has:

```
orders

customers

transactions

products
```

Instead of putting everything together,

organize them as:

```
Sales

↓

bronze

↓

orders_raw

customers_raw

--------------------

silver

↓

orders_clean

customers_clean

--------------------

gold

↓

sales_dashboard
```

Schemas improve organization and simplify permission management.

---

# Catalog vs Schema

| Catalog | Schema |
|----------|---------|
| Top-level container | Container inside a Catalog |
| Groups business domains | Groups related objects |
| Example: Finance | Example: payroll |
| Contains Schemas | Contains Tables, Views, Functions |

---

# Naming Example

```
finance.payroll.employees
```

Meaning:

```
Catalog

↓

finance

Schema

↓

payroll

Table

↓

employees
```

Another example:

```
sales.gold.monthly_revenue
```

```
Catalog

↓

sales

Schema

↓

gold

Table

↓

monthly_revenue
```

---

# Enterprise Design

Large organizations typically separate data like this:

```
Metastore

│

├── Sales
│      ├── bronze
│      ├── silver
│      └── gold
│
├── Finance
│      ├── bronze
│      ├── silver
│      └── gold
│
├── HR
│      ├── bronze
│      ├── silver
│      └── gold
│
└── Marketing
       ├── bronze
       ├── silver
       └── gold
```

This makes governance and access control much easier.

---

# Best Practices

✅ Organize Catalogs by business domain.

Examples:

- Finance
- Sales
- HR
- Marketing

---

✅ Organize Schemas by purpose.

Examples:

- bronze
- silver
- gold

or

- reporting
- staging
- curated

---

✅ Avoid creating too many Catalogs.

Only create new Catalogs when there is a clear business or governance requirement.

---

# Common Mistakes

❌ Creating one Catalog for every project.

❌ Mixing unrelated business domains in the same Catalog.

❌ Storing all tables inside one Schema.

❌ Ignoring a consistent naming convention.

---

# Associate Exam Notes

Remember:

```
Metastore

↓

Catalog

↓

Schema

↓

Table
```

Catalogs organize business domains.

Schemas organize related database objects.

A Catalog contains multiple Schemas.

A Schema contains Tables, Views, and other supported objects.

---

# Professional Interview Questions

## Question 1

A company has Sales, HR, Finance, and Marketing departments.

Should you create one Catalog or four?

### Expected Answer

Create separate Catalogs for each department to simplify governance, permission management, and data ownership.

---

## Question 2

Why shouldn't every table be stored inside one Schema?

### Expected Answer

Using multiple Schemas improves logical organization, simplifies permission management, and makes the environment easier to maintain as the number of tables grows.

---

# Chapter Summary

```
Metastore

↓

Catalog

↓

Schema

↓

Table
```

Think of it as:

```
Drive

↓

Folder

↓

Subfolder

↓

Files
```

---

# Key Takeaways

- A Catalog is the top-level organizational container inside a Metastore.
- A Schema is a logical container within a Catalog.
- Catalogs group business domains.
- Schemas group related database objects.
- Organizing data into Catalogs and Schemas improves governance, security, and discoverability.