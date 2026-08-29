# Retrospect

### Bi-Temporal Enterprise Database Management System

> An insurance database platform built on bi-temporal data management — complete historical tracking, temporal querying, auditing, and regulatory compliance baked into the schema itself.

<div align="center">
  <img src="https://img.shields.io/badge/PostgreSQL-14%2B-336791?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/PL%2FpgSQL-Triggers%20%26%20Procedures-336791?style=for-the-badge&logo=postgresql&logoColor=white" alt="PL/pgSQL"/>
  <img src="https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white" alt="Jupyter"/>
</div>

---

## Table of Contents

1. [The Problem](#the-problem-destructive-updates)
2. [The Solution](#the-solution-bi-temporal-architecture)
3. [Core Features](#core-features)
4. [System Architecture](#system-architecture)
5. [Project Structure](#project-structure)
6. [Getting Started](#getting-started)
7. [Time-Travel Demos](#time-travel-demonstrations)
8. [Testing](#testing--validation)
9. [Documentation](#documentation)
10. [Tech Stack](#tech-stack)

---

## The Problem: Destructive Updates

Standard relational databases overwrite data in place. When a customer moves cities or a policy premium changes, the old value is gone — a regular `UPDATE` or `DELETE` physically destroys whatever was there before.

```
  Customer moves from Delhi -> Mumbai

  BEFORE:  { city: "Delhi",  updated: "2023-01-01" }
  AFTER:   { city: "Mumbai", updated: "2024-06-15" }

  "Delhi" is permanently destroyed. No trace it ever existed.
```

In regulated industries like insurance, this is a serious problem. You can't process retroactive claims, you can't reconstruct what a policy looked like three years ago, and you can't answer basic audit questions like *"what did we have on record on March 3rd, 2022?"*

---

## The Solution: Bi-Temporal Architecture

This project re-engineers the database to track **two independent time dimensions** on every row:

| Dimension | What it tracks | Example |
|:--|:--|:--|
| **Valid Time** | When the fact is true in the real world | "This policy is active Jan 1 – Dec 31" |
| **Transaction Time** | When the system recorded the fact | "Entered by the clerk on Jan 5 at 14:32" |

PostgreSQL triggers intercept every `UPDATE` and `DELETE` at the engine level. Instead of destroying data, the trigger:

1. Archives the old row into a `_history` table with closed timestamps
2. Inserts the new version into the active table with a bumped version number
3. Links both so the full timeline is always reconstructable

The result is an append-only audit log that still behaves like a normal database from the outside.

---

## Core Features

- **Immutable Auditing** — Versioning happens autonomously via triggers. No tracking code needed in the application layer.
- **Time-Travel Queries** — Temporal views and GiST indexes let you query the exact state of the database at any point in the past.
- **Logical Deletes** — `DELETE` removes data from the active table but preserves it permanently in history.
- **Strict Database API** — 14 stored procedures encapsulate all data entry and reject invalid financial states at the engine level.
- **GiST Indexed** — Generalized Search Tree indexes on temporal ranges for fast time-travel lookups.
- **Automated Data Generation** — Python scripts generate 100+ realistic baseline records for testing.

---

## System Architecture

<p align="center">
  <img src="UML diagram.jpg" alt="System UML Diagram" width="720"/>
</p>

```
+------------------------------------------------------------------+
|                        APPLICATION LAYER                          |
|                   (Jupyter Notebook / pgAdmin)                    |
+------------------------------+-----------------------------------+
                               |  SQL Calls
                               v
+------------------------------------------------------------------+
|                     API LAYER (Stored Procedures)                 |
|                                                                   |
|  create_customer()  .  update_customer_address()                  |
|  create_policy()    .  update_policy_premium()                    |
|  register_claim()   .  approve_claim()                            |
|  record_payment()   .  ... (14 total)                             |
|                                                                   |
|  Encapsulates business logic.                                     |
|  RAISE EXCEPTION to rollback invalid transactions.                |
+------------------------------+-----------------------------------+
                               |  DML Operations
                               v
+------------------------------------------------------------------+
|                  AUTOMATION LAYER (PL/pgSQL Triggers)             |
|                                                                   |
|  - Intercepts every UPDATE and DELETE on every table              |
|  - Archives old row into _history table (closed timestamps)       |
|  - Spawns new version in active table (bumped version_number)     |
|  - Flags logical deletes with is_current = FALSE                  |
+------------------------------+-----------------------------------+
                               |
                               v
+------------------------------------------------------------------+
|                       STORAGE LAYER                               |
|                                                                   |
|   Active Tables          History Tables (_history suffix)         |
|        |                        |                                 |
|        +----------+-------------+                                 |
|                   v                                               |
|           UNION ALL Views + GiST Indexes                          |
|           (seamless timeline querying)                             |
+------------------------------------------------------------------+
```

---

## Project Structure

```
Retrospect/
|
|-- Phase 1-Legacy_Simple_DBMS/         # Traditional static database (the "before")
|   |-- ER Diagram.png
|   |-- Insurance database DDL.sql
|   |-- Insurance database DML.sql
|   |-- Insurance database DQL.sql
|   |-- Insurance database DCL.sql
|   |-- Insurance database TCL.sql
|   |-- Insurance database Sample Data.sql
|   |-- Insurance database python code.ipynb
|   +-- README.md
|
|-- Phase 2-Bi-Temporal_DBMS/           # The bi-temporal solution
|   |-- Bi-Temporal ER Diagram.png
|   |-- Insurance database Bi-Temporal DDL.sql
|   |-- Insurance database Bi-Temporal DML.sql
|   |-- Insurance database Bi-Temporal DQL.sql
|   |-- Insurance database Triggers.sql
|   |-- Insurance database Procedures.sql
|   |-- Insurance database Functions.sql
|   |-- Insurance database Temporal Demo.sql
|   |-- Insurance database Temporal Queries.sql
|   |-- Insurance database Python Code.ipynb
|   +-- README.md
|
|-- Phase 3-Comparison/                 # Architectural analysis
|   |-- Feature Comparison.md
|   |-- Legacy vs Bi-Temporal Database.md
|   |-- Performance Comparison.md
|   +-- Query Comparison.md
|
|-- Project Documentation/              # Thesis and guides
|   |-- Official_Documentation_Compiled.pdf
|   |-- Official_Documentation_Compiled.md
|   |-- Software Architecture.md
|   |-- User Guide.md
|   +-- Project README.md
|
|-- UML diagram.jpg
|-- testing_report.md
|-- requirements.txt
+-- README.md
```

---

## Getting Started

### Prerequisites

- **PostgreSQL 14+**
- **Python 3.x** with `pandas`, `sqlalchemy`, `psycopg2-binary`
- **Jupyter Notebook**
- **pgAdmin** (optional, for GUI access)

### Setup

**1. Clone the repo**
```bash
git clone https://github.com/anshul755/Retrospect.git
cd Retrospect
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Create the database**
```sql
CREATE DATABASE insurance_bitemporal;
```

**4. Configure credentials**

Open `Phase 2-Bi-Temporal_DBMS/Insurance database Python Code.ipynb` and update the connection string with your PostgreSQL username and password.

**5. Initialize**

Run the notebook cells in order. `initialize_database()` will set up tables, triggers, procedures, and generate 100 rows of baseline data.

**6. Generate temporal history**

Uncomment and run the `run_temporal_demo()` cell. This simulates real-world changes (address updates, policy renewals, claim processing) and proves the database archives every previous state automatically.

> For a detailed walkthrough, see the [User Guide](Project%20Documentation/User%20Guide.md).

---

## Time-Travel Demonstrations

Open `Phase 2-Bi-Temporal_DBMS/Insurance database Temporal Queries.sql` in pgAdmin and run queries like:

**AS OF Query** — what did the database look like 5 minutes ago?

```sql
SELECT customer_id, first_name, last_name, city
FROM insurance.v_customer_timeline
WHERE (CURRENT_TIMESTAMP - INTERVAL '5 minutes')
      BETWEEN transaction_from AND transaction_to;
```

**Version History** — every change ever made to a customer:

```sql
SELECT version_number, city, transaction_from, transaction_to, is_current
FROM insurance.v_customer_timeline
WHERE customer_id = 1
ORDER BY version_number;
```

**Logical Delete Recovery** — what was deleted and when?

```sql
SELECT *
FROM insurance.customer_history
WHERE is_current = FALSE
  AND operation_type = 'DELETE';
```

---

## Testing & Validation

All tests pass. Full details in [testing_report.md](testing_report.md).

| Test | What it proves | Result |
|:--|:--|:--|
| API Execution | Procedures insert data with `version_number = 1` and infinity end-dates | Pass |
| Temporal Versioning | Triggers archive V1 to history, spawn V2 in active table | Pass |
| Coverage Constraints | Claims exceeding policy coverage are rejected | Pass |
| Payment Validation | Mismatched payments rejected by `payment_check_fn` trigger | Pass |
| Workflow State Mgmt | Claim lifecycle (PENDING to APPROVED) creates linked version timeline | Pass |
| Backward Compatibility | Raw `DELETE` is intercepted; data archived, never destroyed | Pass |

---

## Documentation

| Document | Description |
|:--|:--|
| [User Guide](Project%20Documentation/User%20Guide.md) | Step-by-step walkthrough |
| [Software Architecture](Project%20Documentation/Software%20Architecture.md) | Technical architecture details |
| [Official Documentation (PDF)](Project%20Documentation/Official_Documentation_Compiled.pdf) | Compiled thesis |
| [Feature Comparison](Phase%203-Comparison/Feature%20Comparison.md) | Legacy vs bi-temporal feature matrix |
| [Performance Comparison](Phase%203-Comparison/Performance%20Comparison.md) | Benchmark analysis |
| [Query Comparison](Phase%203-Comparison/Query%20Comparison.md) | Side-by-side SQL comparison |

---

## Tech Stack

| Layer | Technology |
|:--|:--|
| Database | PostgreSQL 14+ with PL/pgSQL |
| Temporal Engine | Custom triggers, stored procedures, GiST-indexed temporal views |
| Data Generation | Python 3, Pandas, SQLAlchemy, psycopg2 |
| Visualization | Plotly, Jupyter Notebook |
| Documentation | Markdown, PDF |
