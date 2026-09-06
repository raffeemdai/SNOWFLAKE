# Snowflake Complete Notes — Zero to Hero

**Theory · Analogies · SQL Code & Examples · Interview Q&A**
Based on the full Cloudlearningyard Snowflake Tutorial Playlist (Videos 1–47, excluding Video 32 and Video 46 which are members-only and unavailable)

---

## Table of Contents

**Part A — Fundamentals & Architecture**
1. [Introduction to Snowflake](#1-introduction-to-snowflake)
2. [Snowflake Architecture — The Three Layers](#2-snowflake-architecture--the-three-layers)
3. [Account Setup & Pricing](#3-account-setup--pricing)
4. [Snowsight UI Walkthrough](#4-snowsight-ui-walkthrough)
5. [Virtual Warehouses](#5-virtual-warehouses)

**Part B — Database Objects, Data Types & Loading**
6. [Databases, Tables & UI Data Load](#6-databases-tables--ui-data-load)
7. [File Formats](#7-file-formats)
8. [Semi-Structured Data Ingestion](#8-semi-structured-data-ingestion)
9. [Table Types (Permanent / Transient / Temporary / External)](#9-table-types-permanent--transient--temporary--external)
10. [Stages](#10-stages)
11. [SnowSQL (CLI)](#11-snowsql-cli)
12. [COPY INTO — Table Ingestion](#12-copy-into--table-ingestion)
13. [COPY INTO — Unloading to Stages](#13-copy-into--unloading-to-stages)

**Part C — Cloud Integrations & Automated Ingestion**
14. [Loading Data from AWS S3](#14-loading-data-from-aws-s3)
15. [Loading Data from Google Cloud Storage (GCS)](#15-loading-data-from-google-cloud-storage-gcs)
16. [Continuous Ingestion Using Snowpipe (AWS)](#16-continuous-ingestion-using-snowpipe-aws)
17. [Snowpipe Error Notifications](#17-snowpipe-error-notifications)
18. [Snowsight Dashboards & KPIs](#18-snowsight-dashboards--kpis)
19. [External Tables](#19-external-tables)
20. [Caching in Snowflake](#20-caching-in-snowflake)

**Part D — Data Protection, Views & Security**
21. [Time Travel & Fail-Safe](#21-time-travel--fail-safe)
22. [Zero-Copy Cloning](#22-zero-copy-cloning)
23. [Views — Standard, Materialized & Secure](#23-views--standard-materialized--secure)
24. [Streams — Change Data Capture (CDC)](#24-streams--change-data-capture-cdc)
25. [Tasks — Automating Workflows](#25-tasks--automating-workflows)
26. [Role-Based Access Control (RBAC)](#26-role-based-access-control-rbac)
27. [Dynamic Data Masking (DDM)](#27-dynamic-data-masking-ddm)
28. [Account Usage vs. Information Schema](#28-account-usage-vs-information-schema)
29. [Flattening Semi-Structured Data (JSON & XML)](#29-flattening-semi-structured-data-json--xml)
30. [Secure Data Sharing](#30-secure-data-sharing)

**Part E — Cost Governance, Advanced Loading & Modern Pipelines**
31. [Resource Monitors](#31-resource-monitors)
32. [Micro-Partitioning & Clustering *(unavailable — members-only)*](#32-micro-partitioning--clustering-unavailable--members-only)
33. [INFER_SCHEMA — Auto-Detect Table Structure](#33-infer_schema--auto-detect-table-structure)
34. [Loading Data from Azure Blob Storage](#34-loading-data-from-azure-blob-storage)
35. [SCD Type 2 Using Streams & Tasks](#35-scd-type-2-using-streams--tasks)
36. [Task Monitoring via AWS SNS](#36-task-monitoring-via-aws-sns)
37. [The CHANGES Clause vs. Streams](#37-the-changes-clause-vs-streams)
38. [Snowflake + GitHub Integration](#38-snowflake--github-integration)
39. [Dynamic Tables](#39-dynamic-tables)
40. [SCD Type 1 & 2 Using Dynamic Tables](#40-scd-type-1--2-using-dynamic-tables)

**Part F — Applications, Governance & Programmability**
41. [Streamlit in Snowflake](#41-streamlit-in-snowflake)
42. [Tag-Based Masking Policies](#42-tag-based-masking-policies)
43. [Loading API Data into Snowflake](#43-loading-api-data-into-snowflake)
44. [Nested JSON — Double/Hierarchical Flattening](#44-nested-json--doublehierarchical-flattening)
45. [Snowpipe on Azure — Auto-Ingest Pipeline](#45-snowpipe-on-azure--auto-ingest-pipeline)
46. [Clustering & Query Acceleration Service *(unavailable — members-only)*](#46-clustering--query-acceleration-service-unavailable--members-only)
47. [Stored Procedures](#47-stored-procedures)

**Part G — Interview Preparation**
48. [Comprehensive Interview Questions & Answers](#48-comprehensive-interview-questions--answers)
49. [Conclusion](#49-conclusion)

---

# Part A — Fundamentals & Architecture

## 1. Introduction to Snowflake

### 1.1 Why Cloud Data Warehouses Emerged

Traditional on-premises databases were built for relatively small, structured data volumes. As data exploded in size and variety (structured **and** semi-structured), organizations faced **data silos** — information scattered across different systems with no unified view. This led to traditional **data warehouses**, which centralized data from heterogeneous sources (operational databases, CRMs, external feeds) for analytics and reporting.

But on-premises data warehouses still had major limitations:
- High infrastructure and maintenance costs
- Limited, hard-to-expand storage capacity
- Poor scalability, failover, and replication support
- Ongoing security and patching burden

This gave rise to **Cloud Data Warehouses** — Snowflake, Redshift, BigQuery — which let organizations store and process huge amounts of data without heavy infrastructure investment, essentially as **Software-as-a-Service (SaaS)**.

### 1.2 What is Snowflake?

Snowflake is a **true SaaS offering**: you don't install any software or hardware, don't configure servers, and don't manage upgrades. You simply create an account and start working.

> ⚠️ **Common Misconception:** Snowflake is *not* a cloud provider like AWS or GCP. Snowflake runs *on top of* a public cloud (AWS, Azure, or GCP) — you choose which one when creating your account — but Snowflake itself is a platform, not the underlying infrastructure. It only runs on **public cloud**, never on-premises or private cloud.

Snowflake is a single platform supporting many workloads: data warehousing, data exchange, data engineering, data lake, data science, and data applications — with (in principle) unlimited scalability. Storing petabytes of data requires no special configuration from the user; only compute sizing requires some thought.

### 1.3 Key Snowflake Advantages

| Advantage | What It Means |
|---|---|
| **Pay-per-use** | You pay for storage always, but compute cost is billed only while a virtual warehouse is actively running |
| **Optimized storage** | Columnar format, automatically compressed, organized into micro-partitions |
| **Caching** | Multiple caching layers boost performance and reduce repeated compute cost |
| **Zero-Copy Cloning** | Instant snapshots of databases/schemas/tables without duplicating storage |
| **Time Travel & Fail-Safe** | Built-in data recovery windows without manual backups |
| **Secure Data Sharing** | Share live data with other accounts without copying it |
| **Native semi-structured support** | JSON, XML, Parquet, Avro, ORC ingested and queried without upfront schema design |
| **Multi-cluster architecture** | Elastic scaling for both raw compute power and concurrency |

> 🏢 **Analogy — Renting a Fully-Serviced Office vs. Owning a Building**
>
> An on-premises data warehouse is like owning an office building: you buy the land, construct it, maintain the plumbing and electricity, and pay for the whole building whether you use 10% or 100% of it. Snowflake is like renting fully-serviced office space in a shared tower: you show up, plug in your laptop, and pay only for the desks (compute) you actually use — while storage (your filing cabinets) is billed separately and modestly. The building's maintenance, security, and upgrades are the landlord's (Snowflake's) problem, not yours.

---

## 2. Snowflake Architecture — The Three Layers

Understanding architecture matters because it lets you diagnose performance problems, understand billing, and use features effectively.

Snowflake describes its own design as a **"multi-cluster, shared data" architecture** that is elastic by nature — a **hybrid** of **shared-disk** and **shared-nothing** architecture:
- **Shared-disk** aspect: a single, central data repository that *all* compute nodes can access.
- **Shared-nothing** aspect: queries are processed using **MPP (Massively Parallel Processing)** — each compute node works on its own local slice of data — giving the performance and scalability benefits of shared-nothing systems while keeping data management simple.

### 2.1 The Three Layers

```
┌─────────────────────────────────────────┐
│      CLOUD SERVICES LAYER (the Brain)    │
│  Auth, Access Control, Metadata,         │
│  Query Optimization, Infrastructure Mgmt │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│   VIRTUAL WAREHOUSE LAYER (Compute)      │
│  Independent compute clusters that       │
│  query, load, and unload data            │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│      DATA STORAGE LAYER (Storage)        │
│  Centralized, compressed, columnar,      │
│  encrypted storage in micro-partitions   │
└─────────────────────────────────────────┘
```

#### 1. Cloud Services Layer — "The Brain"

Responsible for:
- **Authentication and Access Control** — validating login credentials, and controlling exactly which databases/tables/warehouses a user can access based on their role.
- **Infrastructure & Metadata Management** — tracking table names, row counts, micro-partition counts, min/max values, etc.
- **Query Parsing & Optimization** — compiling and optimizing SQL before execution.
- **Data Sharing & Security** — governs secure sharing and enforces security policies.

> 🛂 **Analogy — The Security Guard at a New Office**
>
> Imagine joining a new company. The security guard at the gate (authentication) checks your ID card before letting you in — matching your credentials against what HR authorized. Once inside, the receptionist (access control) only lets you into rooms you're authorized for — not every meeting room. The Cloud Services layer works exactly this way: your username/password is validated (authentication), and then your assigned role determines exactly which databases, schemas, and tables you can actually touch (authorization/access control). Try to `SELECT * FROM customer` on a table you don't have rights to, and you'll get an error — either "object does not exist" or "insufficient privileges" — just like being turned away at a locked door.

#### 2. Virtual Warehouse Layer — "Compute"

This is **not** a data warehouse in the traditional sense — it's a **compute engine** (CPU + memory + temporary SSD) used to load, unload, and query data. Under the hood it's simply a cluster of cloud VM instances (EC2 on AWS, Azure VMs, or Google Compute VMs), but Snowflake never discloses the exact underlying configuration.

Key property: **multiple virtual warehouses are completely independent of each other** — you can scale, suspend, or resize one warehouse without impacting any other warehouse's performance. This separation of storage and compute is one of Snowflake's defining architectural decisions.

#### 3. Data Storage Layer — "Storage"

All data — structured or semi-structured — lives here, in a **centralized, compressed, hybrid-columnar format**. Data is automatically broken into **micro-partitions** (50–500 MB chunks). The physical storage (e.g., AWS S3, Azure Blob, GCP Cloud Storage) is entirely managed by Snowflake — you cannot see or directly access these underlying files, even if you already have your own account with that cloud provider. Everything is encrypted using **AES-256** encryption, and this is fully managed by Snowflake — no customer configuration required.

### 2.2 Query Execution Flow (Putting the Layers Together)

When you run `SELECT * FROM customer`:

1. **Cloud Services layer** validates your identity and checks your access rights to the `customer` table.
2. If authorized, the query moves to the **Virtual Warehouse layer**, which wakes up (if suspended) and executes the query — this is where you see the query turn from grey to green in Snowsight, and where "query provisioning" time (warehouse wake-up) is counted.
3. Data is pulled from the **Data Storage layer** as needed.
4. Results are returned, and a **unique Query ID** is generated for every single execution — even re-running the exact same statement produces a brand-new Query ID.

Every query's profile (visible via the Query ID) shows compilation time, provisioning time, execution time, rows scanned, and caching behavior.

---

## 3. Account Setup & Pricing

### 3.1 Creating an Account

1. Go to `snowflake.com`, click **Start for Free**.
2. Fill in basic details (name, email, company).
3. You get a **30-day free trial** with roughly **$400** in credits.
4. Choose your **Edition** (Standard, Enterprise, Business Critical, or Virtual Private Snowflake — the last requiring direct sales contact) and your **Cloud Provider + Region** (AWS, Azure, or GCP).
5. You'll receive an activation email — click **"Click to Activate"**, set your username/password, and you're in.

> 💡 Enterprise Edition is commonly used for learning because it unlocks features like multi-cluster warehousing, materialized views, and longer Time Travel windows that Standard Edition doesn't support.

### 3.2 Snowflake Pricing — Two Independent Components

Snowflake bills **storage** and **compute** completely **separately**:

| Component | Approx. Cost | Notes |
|---|---|---|
| **Storage** | ~$23–25 per TB per month | Varies by cloud provider and region (e.g., AWS US East ≈ $23/TB, EU regions can be higher) |
| **Compute** | Priced in **Credits** — $2/credit (Standard), $3/credit (Enterprise), $4/credit (Business Critical) | Also varies by region/cloud; e.g., EU pricing is higher than US |

#### Credits and Warehouse Size

Credit consumption is a direct function of **warehouse size** (T-shirt sizing) and **how long it runs**:

| Size | Servers | Credits/hour |
|---|---|---|
| XS (Extra Small) | 1 | 1 |
| S (Small) | 2 | 2 |
| M (Medium) | 4 | 4 |
| L (Large) | 8 | 8 |
| XL | 16 | 16 |
| 2XL | 32 | 32 |
| 3XL | 64 | 64 |
| 4XL | 128 | 128 |
| 5XL | 256 | 256 |
| 6XL | 512 | 512 |

So on Enterprise Edition (US, $3/credit), an Extra Small warehouse running for one hour costs **$3/hour**; a Small costs **$6/hour**; and so on — multiply credits by the per-credit dollar rate.

#### Regional Price Variance — A Concrete Example

Both compute and storage pricing genuinely shift by cloud provider and region — this isn't just a rounding difference. For example, on Enterprise Edition, AWS US East pricing (~$3/credit) can rise to **~$3.90/credit on AWS EU regions**, and storage that costs **~$23/TB/month on AWS US East** can rise to **~$24.50/TB/month on AWS EU (Frankfurt)**. Always check the official Snowflake pricing page for your specific cloud + region combination before estimating a project's cost.

#### Billing Granularity

- Snowflake bills **per second**, **but** enforces a **minimum billing of 60 seconds** per warehouse activation. A query that finishes in 42 seconds is still billed for a full 60 seconds; a query running for 450 seconds is billed for exactly 450 seconds.
- **DDL statements** (like `CREATE TABLE`) and simple **metadata-only queries** (like `SELECT COUNT(*)` on a structured table) do **not require an active virtual warehouse** — they're resolved entirely by the Cloud Services layer, so they don't consume compute credits.

> ☎️ **Analogy — Old vs. New Mobile Call Plans**
>
> Older prepaid mobile plans charged a flat ₹1 per call regardless of whether you talked for 5 seconds or 59 seconds — a **minimum billing unit**. Later plans switched to true per-second billing after that first unit. Snowflake's compute billing works the same way: there's a minimum 60-second charge (like the old flat per-call charge), and beyond that, it's pure per-second billing (like the newer plans) — no waste, but also no free lunch for very short queries.

---

## 4. Snowsight UI Walkthrough

Snowsight is Snowflake's modern web UI (replacing the older "Classic UI").

### 4.1 The Homepage

Key sections you'll use:
- **Home** — a **universal search bar** that can find any object (tables, views, stored procedures) across your account, or even jump straight to Snowflake documentation.
- **Quick Actions** — shortcuts like Query Data, Create Warehouse, Create User, Upload Local File.
- **+ Create** — create SQL worksheets, Python worksheets, notebooks, or folders.
- **Projects** — where your Worksheets, Notebooks, Streamlit apps, and Dashboards live.
- **Database** — browse databases, schemas, tables you have access to.
- **Monitoring** — Query History, Copy History, Task History, Dynamic Table History.
- **AI/ML Studio** — for ML-related work.
- **Admin** — warehouses, resource monitors, users/roles, billing — typically restricted to senior/admin roles.

### 4.2 Worksheets — Where You'll Spend Most of Your Time

Before running any query, you set the **context**: Database, Schema, Role, and Warehouse. Once set, you can run standard SQL — for example, creating your very first database and schema straight from a worksheet:

```sql
-- Creating a database automatically creates PUBLIC + INFORMATION_SCHEMA underneath it
CREATE DATABASE youtube_training;

-- Creating an additional, custom schema inside that database
CREATE SCHEMA raw_data;
```

Run a highlighted statement with `Ctrl+Enter` (or the blue "run" icon) — the query pane below will show the result set, and a Query ID/status pane will confirm success or failure.

Key worksheet features:
- **Query Profiling** — every executed query shows a Query ID, execution time breakdown (compilation, provisioning, execution), rows scanned, and whether caching was used.
- **Run / Run All** — run a single highlighted statement (`Ctrl+Enter`) or all statements in the worksheet (`Ctrl+Shift+Enter`).
- **Result Pane Controls** — hide/show results, select specific columns to display, download results as CSV/TSV.
- **Query History (clock icon)** — every query run in that worksheet, filterable by status (queued, running, failed, successful). Beyond the obvious columns (warehouse, user, status, query ID, duration, start time), you can also surface extra columns like **Session ID, Client Driver, Rows, Query Tag,** and **Incident** for deeper diagnostics.
- **Code Versions** — every time you *run* a statement, a new "version" of the worksheet's code is saved (identical reruns don't create duplicate versions). This lets you recover code you accidentally overwrote or deleted — a built-in undo/version-history mechanism.
- **Format Query** — auto-formats messy SQL into clean, readable SQL.
- **Sharing** — worksheets can be shared with teammates with granular permission (e.g., "View and Run" vs. "View Results only" vs. cannot view), depending on the invitee's assigned role.
- **Folders** — organize multiple worksheets into folders (right-click → Create Folder).

### 4.3 Other Worksheet Types

- **Python Worksheets** — write **Snowpark** code directly.
- **Notebooks** — similar to Jupyter/Databricks notebooks; mix Python and SQL cells, and even pass data between cells.
- **Streamlit** — build full interactive apps (see Section 41).
- **Dashboards** — lightweight visualizations (not as advanced as Power BI/Tableau, but sufficient for account health/usage stats).

### 4.4 Monitoring Section

- **Query History** — last 14 days of all queries run in the account (not just by you), filterable by user, warehouse, status.
- **Copy History** — success/failure status of every `COPY INTO` operation.
- **Task History**, **Dynamic Table History** — execution logs for scheduled tasks and dynamic table refreshes.

---

## 5. Virtual Warehouses

A **Virtual Warehouse** is Snowflake's **compute engine** — not a data storage repository. It powers:
- Running `SELECT` queries (simple or complex)
- Loading data (`COPY INTO <table>`) and unloading data (`COPY INTO <location>`)

### 5.1 What Doesn't Need a Warehouse

**DDL statements** (`CREATE TABLE`, etc.) and simple **metadata-only queries** (e.g., `SELECT COUNT(*)`) are resolved entirely by the **Cloud Services layer** — no active warehouse required, and no compute cost incurred.

### 5.2 Scaling — Vertical vs. Horizontal

| Scaling Type | What It Does | When to Use |
|---|---|---|
| **Vertical (Scale Up/Down)** | Change warehouse *size* (XS → M → L, etc.) | Speed up a single large, complex query by giving it more raw CPU/RAM |
| **Horizontal (Scale Out/In)** | Add/remove **clusters** of the *same size* warehouse (Multi-cluster Warehouses) | Handle heavy **concurrency** — many simultaneous users/queries — preventing queueing |

> 🏋️ **Analogy — Bigger Truck vs. More Trucks**
>
> Vertical scaling is like swapping a small delivery van for a bigger truck to carry one giant shipment faster. Horizontal scaling is like adding more vans of the same size to your fleet because you suddenly have many separate deliveries happening at once — each van handles its own delivery independently, none of them individually needs to be bigger.

### 5.3 Auto-Suspend & Auto-Resume

- **Auto-Suspend** — automatically suspends the warehouse after a period of inactivity (e.g., 5 minutes) to stop wasting credits.
- **Auto-Resume** — automatically wakes the warehouse the instant a new query needs it.

### 5.4 Scaling Policies for Multi-Cluster Warehouses

| Policy | Behavior | Priority |
|---|---|---|
| **Standard** | Waits only ~20 seconds of queued load before spinning up an additional cluster | Performance-first |
| **Economy** | Waits ~6 minutes of *sustained* high load before adding a cluster | Cost-first |

### 5.5 Cost vs. Performance Trade-off

Sometimes a **bigger** warehouse is actually **cheaper overall** for a heavy query: if a Large warehouse finishes a job **4× faster** than a Small warehouse, the total credit consumption can end up roughly the same — but you save significant wall-clock time. Choosing the right size for your workload is one of the most important skills for anyone managing Snowflake cost and performance (especially relevant for people with 8+ years of experience who may take on warehouse-sizing responsibilities).

### 5.6 Creating a Warehouse via SQL

While warehouses are most often created through the Snowsight "+ Warehouse" wizard, the same thing can be done with plain SQL — useful for scripting and CI/CD:

```sql
CREATE OR REPLACE WAREHOUSE my_wh
  WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 300         -- seconds of inactivity (5 minutes) before suspending
  AUTO_RESUME = TRUE         -- automatically wake up on the next incoming query
  MIN_CLUSTER_COUNT = 1      -- multi-cluster: minimum clusters to keep running
  MAX_CLUSTER_COUNT = 3      -- multi-cluster: maximum clusters allowed to spin up
  SCALING_POLICY = 'STANDARD'; -- or 'ECONOMY'
```

`MIN_CLUSTER_COUNT` / `MAX_CLUSTER_COUNT` greater than 1 is what actually turns this into a **Multi-cluster Warehouse** capable of horizontal scaling (Section 5.2); leaving both at `1` keeps it a single-cluster warehouse that can only scale vertically by resizing.

---

# Part B — Database Objects, Data Types & Loading

## 6. Databases, Tables & UI Data Load

### 6.1 Default Schemas

Every new database automatically gets two schemas:
- **`INFORMATION_SCHEMA`** — read-only metadata about all objects in that database.
- **`PUBLIC`** — the default writable schema for your objects.

### 6.2 Data Types & Length Behavior

If you declare a `VARCHAR` column without a length (e.g., `ADDRESS VARCHAR`), Snowflake silently defaults to the **maximum supported size: 16,777,216 characters (~16MB)**. Unlike on-prem databases, there's **no storage or performance penalty** for this — Snowflake dynamically compresses storage based on actual content, not declared size.

### 6.3 Constraints — The Non-Enforcement Rule ⚠️

This is one of the most commonly asked interview topics:

- Snowflake lets you declare `PRIMARY KEY`, `FOREIGN KEY`, and `UNIQUE` constraints — but **only for documentation/compatibility purposes with BI tools and data modeling**.
- **Snowflake does NOT enforce these constraints during DML.** You *can* insert duplicate primary keys successfully.
- The **only constraint Snowflake strictly enforces is `NOT NULL`.** Attempting to insert a `NULL` into a `NOT NULL` column throws an immediate error.

> 🪧 **Analogy — A Sign vs. a Locked Door**
>
> A `PRIMARY KEY` in Snowflake is like a "No Entry" sign on a door that's never actually locked — it documents intent, and well-behaved visitors respect it, but nothing physically stops someone from walking through. `NOT NULL`, on the other hand, is an actual locked door — Snowflake genuinely blocks the action.

### 6.4 Four Ways to Load Data

| Method | Description |
|---|---|
| **Snowsight UI** | Simple drag-and-drop upload for files up to 250MB, with automatic schema inference |
| **SnowSQL (CLI)** | `PUT` + `COPY INTO` for local files, including very large files/folders |
| **Snowpipe** | Continuous, event-driven auto-ingestion pipeline |
| **Third-Party ETL/ELT & Code** | Tools like Matillion, Informatica, or custom Python scripts |

### 6.5 UI Data Load Flow

1. Click **"Add Data" → "Load Data into a Table"**.
2. Browse and select a local file (CSV, JSON, XML, etc.).
3. Choose the destination database, schema, and table (new or existing).
4. Snowflake runs **schema inference** automatically — analyzing headers and inferring column data types.
5. Behind the scenes, Snowflake generates a script that creates a temporary file format, stages the file temporarily, and runs a `COPY INTO` command to load the parsed rows.

---

## 7. File Formats

A **File Format** is a reusable, database-level object that tells Snowflake exactly how to *interpret* a raw file (CSV, JSON, XML, Avro, ORC, Parquet) during ingestion (`COPY INTO`) or extraction. Getting this configuration wrong is the #1 cause of failed or garbled data loads.

### 7.1 Critical CSV Parameters

| Parameter | Purpose |
|---|---|
| `TYPE = CSV` | Mandatory — declares the file structure |
| `SKIP_HEADER = <n>` | Skip `n` rows at the top of the file (usually `1` to skip column headers) |
| `FIELD_DELIMITER = '<char>'` | Column separator (comma, pipe, tab, etc.) |
| `FIELD_OPTIONALLY_ENCLOSED_BY = '"'` | Wraps values containing the delimiter (e.g., `"Delhi, India"`) so the parser doesn't split them into extra columns |
| `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` | Allows loading even when the file's column count doesn't match the table's |
| `DATE_FORMAT = 'DD/MM/YYYY'` | Overrides Snowflake's default `YYYY-MM-DD` parsing for custom date layouts |

### 7.2 Common Troubleshooting Scenarios

| Problem | Symptom | Fix |
|---|---|---|
| **Row Splitting** | An address like `"Delhi, India"` gets split into two columns because of the internal comma | Set `FIELD_OPTIONALLY_ENCLOSED_BY = '"'` |
| **Column Count Mismatch** | File has 9 columns, target table has 8 | Set `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` |
| **Date Unrecognized** | File has `17/08/2024`, Snowflake expects `YYYY-MM-DD` | Set `DATE_FORMAT = 'DD/MM/YYYY'` |

---

## 8. Semi-Structured Data Ingestion

Snowflake natively ingests semi-structured data — **JSON, XML, Parquet** — without requiring you to pre-define a rigid schema, using the special **`VARIANT`** data type.

### 8.1 The `VARIANT` Data Type

A Snowflake-proprietary type that can hold up to **16MB** of arbitrary semi-structured content — an entire JSON object, XML document, or array — inside a single column, with no upfront field parsing required.

### 8.2 Ingestion Workflow

```sql
-- 1. Create a table with a single VARIANT column
CREATE TABLE json_table (data_col VARIANT);

-- 2. Create a matching file format
CREATE FILE FORMAT my_json_format TYPE = JSON;
-- (or TYPE = XML, or TYPE = PARQUET)

-- 3. Load via UI or COPY INTO using this file format
```

Once loaded, the entire JSON/XML document sits in the `VARIANT` column as a collapsible hierarchy you can browse in Snowsight.

### 8.3 Querying VARIANT Data

Use **colon/dot path notation** to traverse nested keys, e.g. `data_col:customer:name`. To transform arrays and nested elements into proper relational rows/columns, use `FLATTEN` / `LATERAL FLATTEN` (covered fully in Section 29).

### 8.4 Parquet Needs a Stage

Parquet files are typically loaded via a **table stage** or **named internal stage** as an intermediate landing zone before the relational `COPY INTO` step.

---

## 9. Table Types (Permanent / Transient / Temporary / External)

| Table Type | Persistence | Time Travel | Fail-Safe | Best Use Case |
|---|---|---|---|---|
| **Permanent** (default) | Until dropped | 0–90 days (configurable, default 1 day) | 7 days (fixed) | Long-term critical business data, dimensions, facts, master tables |
| **Transient** | Until dropped | 0–1 day (max 1) | 0 days | ETL/ELT staging tables, raw landing tables — data re-ingestible from source |
| **Temporary** | Dropped when session ends | 0–1 day (max 1) | 0 days | Session-only calculations, intermediate transformations, caching |
| **External** | Read-only pointer; real data lives outside Snowflake | 0 days | 0 days | Querying large (100GB+) cloud-storage data lakes directly without duplicating storage; no DML allowed |

### 9.1 Behavioral Precedence — The Non-Collision Rule

You **can** create a Temporary table with the **same name** as an existing Permanent/Transient table in the same schema. Whenever that name is queried or targeted (even `DROP TABLE customer`), **the Temporary table always takes precedence**. Once the temporary table is dropped (or the session ends), operations automatically fall back to the original Permanent/Transient table.

### 9.2 Cloning Limitations

- Transient and Temporary tables **cannot be cloned into a Permanent table**.
- Permanent tables **can** be cloned into Transient, Temporary, or other Permanent tables.

### 9.3 Fail-Safe Cost Caveat

Fail-Safe (7 days, non-configurable, Permanent-tables-only) is **not free**. If you're changing 1GB of data daily in a Permanent table, multiple historical versions accumulate in Fail-Safe, exponentially increasing your storage bill. For high-churn staging tables, use **Transient** tables instead to eliminate this cost entirely.

### 9.4 Creating Each Table Type — Syntax

```sql
-- Permanent (default — no keyword needed)
CREATE TABLE dim_customer (customer_id INT, name VARCHAR);

-- Transient — ideal for staging/landing tables
CREATE TRANSIENT TABLE staging_orders (order_id INT, raw_payload VARIANT);

-- Temporary — exists only for the current session
CREATE TEMPORARY TABLE session_calc (calc_id INT, result FLOAT);

-- External — a read-only pointer to files already sitting in a stage
CREATE EXTERNAL TABLE ext_orders
  LOCATION = @my_stage
  FILE_FORMAT = (TYPE = CSV);
```

---

## 10. Stages

A **Stage** is an intermediate landing zone — really a **pointer/metadata object** — where files sit before being copied into a table (or unloaded from a table into external storage).

### 10.1 Types of Stages

| Stage Type | Description |
|---|---|
| **User Stage (`@~`)** | Auto-created per user; personal uploads only; cannot be dropped/altered |
| **Table Stage (`@%table_name`)** | Auto-created per table; accessible only by the table owner; no transformations allowed during copy; cannot be dropped/altered |
| **Named Internal Stage (`@stage_name`)** | User-created (`CREATE STAGE`), reusable, highly configurable — supports directory tables, custom file formats, and multi-user access |
| **External Stage** | Points to files in AWS S3, Azure Blob/ADLS, or GCP Storage; created via `CREATE STAGE` with a URL and credentials/integration |

### 10.2 Listing & Loading Files

```sql
-- Listing files (user, table, and named stages)
LIST @~;
LIST @%orders;
LIST @my_stage;
```

Only **Named stages** show up in `SHOW STAGES` — user and table stages don't.

Files are loaded to internal stages using **`PUT`** (via SnowSQL CLI only), or uploaded via the Snowsight UI's "Load files into a Stage" wizard (up to 250MB).

### 10.3 Querying Staged Files Directly

```sql
-- Query positional columns from a staged file before loading into a table
SELECT $1, $2, $3 FROM @my_stage;
```

Associating a file format with the stage lets this positional query correctly skip headers and parse fields:

```sql
CREATE STAGE my_stage FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1);
```

---

## 11. SnowSQL (CLI)

**SnowSQL** is Snowflake's command-line client — used for batch/interactive SQL execution, data loading (`PUT`), and data unloading (`GET`). Available on Windows, macOS, and Linux.

### 11.1 Setup & Connecting

```bash
# Verify install
snowsql -v

# Connect
snowsql -a <account_identifier> -u <username>

# Connect with full context
snowsql -a <account_identifier> -u <username> -d <database_name> -s <schema_name> -w <warehouse_name>
```

The `account_identifier` is extracted from your Snowsight URL — the part between `https://` and `.snowflakecomputing.com`.

### 11.2 PUT & GET — CLI-Only Commands

`PUT` and `GET` **cannot be run from the browser UI** — they must run via SnowSQL.

```sql
-- PUT: local file → internal stage
PUT file://C:/path/to/file.csv @STG_CSV;

-- GET: internal stage → local machine
GET @STG_CSV file://C:/path/to/destination/;

-- Batch upload using wildcards
PUT file://C:/path/to/*.csv @STG_CSV;

-- Force re-upload even if the file was already ingested before
PUT file://C:/path/to/file.csv @STG_CSV OVERWRITE = TRUE;

-- Remove a file from a stage
REMOVE @STG_CSV/file.csv;
```

Files are automatically Gzip-compressed on upload unless overridden.

---

## 12. COPY INTO — Table Ingestion

`COPY INTO <table>` moves staged files persistently into an existing table.

### 12.1 Error Handling & Validation

```sql
-- Dry-run: list every row/column that would fail, without loading anything
COPY INTO dim_customer FROM @STG_CSV
VALIDATION_MODE = RETURN_ALL_ERRORS;
```

| `ON_ERROR` Setting | Behavior |
|---|---|
| `ABORT_STATEMENT` (default) | Any error aborts the entire load |
| `CONTINUE` | Loads all clean rows, skips corrupted ones |
| `SKIP_FILE` | Skips the *entire file* if even one row errors |

```sql
-- Load everything that parses correctly; silently skip bad rows
COPY INTO dim_customer FROM @STG_CSV ON_ERROR = CONTINUE;

-- Discard the whole file the moment a single row fails to parse
COPY INTO dim_customer FROM @STG_CSV ON_ERROR = SKIP_FILE;
```

### 12.2 The 64-Day Metadata Cache & FORCE

Snowflake tracks ingested files in a **64-day metadata cache**. Re-running `COPY INTO` on an already-loaded file **does nothing** by default (it's silently skipped) — unless you override with:

```sql
COPY INTO dim_customer FROM @STG_CSV FORCE = TRUE;
```

### 12.3 Auto-Purging Staged Files

```sql
COPY INTO dim_customer FROM @STG_CSV PURGE = TRUE;
```
Automatically deletes the source file from the stage after a successful load.

### 12.4 Matching Columns by Name (Schema Evolution)

```sql
COPY INTO dim_customer FROM @STG_CSV
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```
Matches source columns by **header name** rather than positional order — useful when file columns arrive shuffled. Requires `PARSE_HEADER = TRUE` in the file format.

### 12.5 Injecting Metadata Columns

```sql
COPY INTO target_table (id, name, source_file, load_time)
FROM (SELECT $1, $2, METADATA$FILENAME, CURRENT_TIMESTAMP() FROM @STG_CSV);
```
`METADATA$FILENAME` and `METADATA$FILE_ROW_NUMBER` let you write audit trails directly during ingestion.

---

## 13. COPY INTO — Unloading to Stages

`COPY INTO <location>` performs the reverse: exporting table data **out** to a stage (internal or external).

```sql
COPY INTO @STG_CSV/customer_export/ FROM customer;
```

### 13.1 Key Unload Parameters

| Parameter | Purpose |
|---|---|
| `SINGLE = TRUE / FALSE` | `FALSE` (default) unloads in parallel into multiple numbered files (`data_0_0_0.csv.gz`); `TRUE` forces a single output file |
| `MAX_FILE_SIZE` | Byte limit before splitting output files (default 16MB, up to 5GB for external stages) |
| `OVERWRITE = TRUE` | Overwrite existing files of the same name in the stage |
| `INCLUDE_QUERY_ID = TRUE` | Appends the unique Query ID to exported filenames |
| `DETAILED_OUTPUT = TRUE` | Returns filenames, row counts, and file sizes in the result pane |
| `HEADER = TRUE` | Writes column names as the first row of the exported file |

### 13.2 Validation Mode for Unloads

```sql
COPY INTO @STG_CSV/Megan_Fox/ FROM customer VALIDATION_MODE = RETURN_ROWS;
```

### 13.3 Putting the Unload Parameters Together

A realistic unload combines several of the parameters above in one statement:

```sql
COPY INTO @STG_CSV/customer_export/
FROM customer
FILE_FORMAT = (TYPE = CSV)
SINGLE = FALSE                -- parallel unload into multiple numbered files (default)
MAX_FILE_SIZE = 5368709120    -- 5GB max per file (external stage upper limit)
OVERWRITE = TRUE              -- replace any existing files of the same name
INCLUDE_QUERY_ID = TRUE       -- stamp the unique Query ID into each output filename
DETAILED_OUTPUT = TRUE        -- return filenames, row counts, and file sizes
HEADER = TRUE;                -- write column names as the first row
```

### 13.4 Copying Files Between Stages Directly

```sql
COPY FILES INTO @STG_DATA FROM @STG_CSV;
```
Moves files between stages without downloading them locally.

---

# Part C — Cloud Integrations & Automated Ingestion

## 14. Loading Data from AWS S3

### 14.1 Insecure vs. Secure Methods

- **Insecure (legacy):** hard-coding AWS Access Key + Secret Key directly in the stage definition — **not recommended**.
- **Secure (recommended):** using **IAM Role delegation** via a **Storage Integration** object — no keys ever stored in Snowflake.

### 14.2 Step-by-Step Secure Integration

```sql
-- Step 1: Create the Storage Integration
CREATE OR REPLACE STORAGE INTEGRATION s3_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/AWS_Data_Ingestion'
  STORAGE_ALLOWED_LOCATIONS = ('s3://youtube-learning-snowflake/source_folder/');

-- Step 2: Retrieve Snowflake's generated IAM User ARN and External ID
DESCRIBE INTEGRATION s3_integration;
-- Look for STORAGE_AWS_IAM_USER_ARN and STORAGE_AWS_EXTERNAL_ID
```

**Step 3:** In the AWS Console, edit the IAM Role's **Trust Relationship** policy to allow that Snowflake IAM user to assume the role, matching the external ID:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::0246813579:user/abc12345" },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": { "sts:ExternalId": "abcde12345_SF_ID" }
      }
    }
  ]
}
```

```sql
-- Step 4: Create the External Stage linked to the integration
CREATE OR REPLACE STAGE aws_s3_stage
  URL = 's3://youtube-learning-snowflake/source_folder/'
  STORAGE_INTEGRATION = s3_integration
  FILE_FORMAT = ingest_data;

-- Step 5: Verify and ingest
LIST @aws_s3_stage;
COPY INTO target_table FROM @aws_s3_stage;
```

> ⚠️ **Removal Warning:** `REMOVE @aws_s3_stage/file.csv` deletes the file from **both** the Snowflake stage pointer **and** the actual S3 bucket, if the IAM role has `S3:DeleteObject` permission. Be careful — this is a real, physical delete, not just a metadata unlink.

---

## 15. Loading Data from Google Cloud Storage (GCS)

### 15.1 Storage Integration (No Role ARN Needed Upfront)

Unlike AWS, GCS integration does **not** require a Role ARN at creation time:

```sql
CREATE OR REPLACE STORAGE INTEGRATION gcs_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'GCS'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://snowflake-youtube-learning-01/source_folder/');
```

### 15.2 Retrieve the Snowflake-Generated Service Account

```sql
DESCRIBE INTEGRATION gcs_integration;
-- Copy STORAGE_GCP_SERVICE_ACCOUNT, e.g.:
-- service-account-id@gcp-snowflake-integration.iam.gserviceaccount.com
```

### 15.3 GCP-Side IAM Configuration

1. Create an IAM Role (e.g., `GCS_Snowflake_Viewer`) with:
   - `storage.objects.get`
   - `storage.objects.list`
   - `storage.objects.create` (only if unloading or using `PURGE = TRUE`)
   - `storage.objects.delete` (only if using `PURGE = TRUE`)
2. In the GCS Bucket's permissions, click **Grant Access**, paste the Snowflake service account as the principal, and assign that role.

### 15.4 Stage & Ingestion

```sql
CREATE OR REPLACE STAGE gcs_stage
  URL = 'gcs://snowflake-youtube-learning-01/source_folder/'
  STORAGE_INTEGRATION = gcs_integration
  FILE_FORMAT = ingest_data;

LIST @gcs_stage;
-- then COPY INTO as usual
```

---

## 16. Continuous Ingestion Using Snowpipe (AWS)

**Snowpipe** is Snowflake's **continuous, serverless** loading service. It detects new files landing in an S3 stage and triggers ingestion **automatically**, in near real-time, without an active virtual warehouse — you pay only for the small serverless compute used to parse the files.

### 16.1 Defining a Pipe

```sql
CREATE OR REPLACE PIPE aws_pipe
  AUTO_INGEST = TRUE
AS
  COPY INTO aws_customer_load
  FROM @aws_s3_stage;
```

### 16.2 Wiring Up the S3 → Snowflake Trigger (SQS)

1. `SHOW PIPES;` → copy the `notification_channel` value (an AWS SQS queue ARN Snowflake automatically manages).
2. In your S3 bucket's **Properties → Create Event Notification**.
3. Set event type to **"All object create events"**.
4. Destination: **SQS queue** → paste in the ARN from step 1.

### 16.3 Health Checks & Monitoring

```sql
-- Check pipe status (pending files, last ingested file, etc.)
SELECT SYSTEM$PIPE_STATUS('aws_pipe');
```
This returns a JSON payload — key fields to inspect include `pendingFileCount` (files queued but not yet ingested), `lastIngestedFilePath` (most recently loaded file), and `lastReceivedMessageTimestamp` (when the last SQS notification arrived) — useful for diagnosing a pipe that appears "stuck."

```sql
-- Serverless credit usage over the last 7 days
SELECT *
FROM TABLE(information_schema.pipe_usage_history(
  date_range_start => DATEADD('day', -7, CURRENT_DATE()),
  pipe_name => 'aws_pipe'
));
```

---

## 17. Snowpipe Error Notifications

Instead of manually checking whether Snowpipe loads succeeded, you can route ingestion failures to a cloud notification system (AWS SNS, Azure Event Grid, GCP Pub/Sub).

### 17.1 AWS SNS Setup Steps

1. **Create an SNS Topic** (e.g., `snowpipe_alerts`).
2. **Add an email subscription** → confirm it from your inbox.
3. **Create an IAM Policy** allowing `sns:Publish` on the topic's ARN.
4. **Attach the policy** to your existing integration role (e.g., `AWS_Data_Ingestion`).
5. **Create a Notification Integration in Snowflake:**

```sql
CREATE OR REPLACE NOTIFICATION INTEGRATION sns_error_integration
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = AWS_SNS
  ENABLED = TRUE
  DIRECTION = OUTBOUND
  AWS_SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789012:snowpipe_alerts'
  AWS_SNS_ROLE_ARN = 'arn:aws:iam::123456789012:role/AWS_Data_Ingestion';
```

6. **Describe & establish handshake:** `DESCRIBE INTEGRATION sns_error_integration;` retrieves an IAM user ARN and external ID — update your AWS trust relationship policy accordingly (same pattern as the S3 storage integration handshake in Section 14).
7. **Attach the error integration to the pipe:**

```sql
ALTER PIPE aws_pipe SET ERROR_INTEGRATION = sns_error_integration;
```

Now, dropping a corrupted file (bad schema, wrong data types) into S3 fails the Snowpipe load and triggers an **email alert** with the exact database, schema, pipe name, filename, and parsing error.

---

## 18. Snowsight Dashboards & KPIs

### 18.1 Information Schema vs. Account Usage — When to Use Which for Dashboards

| Source | Latency | Retention | Best For |
|---|---|---|---|
| **Information Schema** | 0 latency (real-time) | 3–6 months | Immediate, database-local metrics |
| **`snowflake.account_usage`** | 45 min – 3 hours | Up to 365 days | Long-term, account-wide administrative dashboards |

### 18.2 Sample KPI Queries

```sql
-- Total distinct warehouses
SELECT COUNT(DISTINCT warehouse_name)
FROM snowflake.account_usage.warehouse_metering_history;

-- Total storage footprint (active + fail-safe + staged)
SELECT AVG(active_bytes + stage_bytes + failsafe_bytes) / (1024*1024) AS total_mb
FROM snowflake.account_usage.storage_usage;
```

### 18.3 Interactive Filters

Dashboards support dynamic dropdown filters, matched into queries using a colon-prefixed variable:

```sql
SELECT *
FROM snowflake.account_usage.query_history
WHERE warehouse_name = :warehouse;
```

Dashboards can be shared with teammates with fine-grained permissions ("View and Run" vs. "View Results only"), exactly like worksheet sharing.

---

## 19. External Tables

An **External Table** lets you query files sitting in an external cloud stage **directly**, without physically moving them into Snowflake.

### 19.1 Key Properties

- **Entirely read-only** — no `INSERT`/`UPDATE`/`DELETE`.
- Only file-level metadata (filenames, versions, paths) lives in Snowflake — the actual raw data stays in the cloud data lake (S3/GCS/ADLS).
- **Zero Snowflake storage cost** — you only pay the external cloud provider's storage rate.

### 19.2 Two Ways to Define Columns

**A. Without a pre-defined schema** — everything lands in a single `VALUE` variant column:

```sql
CREATE OR REPLACE EXTERNAL TABLE ext_customer_raw
  LOCATION = @aws_s3_stage
  FILE_FORMAT = (TYPE = CSV);

SELECT VALUE:c1::INT AS customer_id, VALUE:c2::VARCHAR AS name FROM ext_customer_raw;
```

**B. With a pre-defined, strongly-typed schema:**

```sql
CREATE OR REPLACE EXTERNAL TABLE ext_customer (
  customer_id INT AS (VALUE:c1::INT),
  name VARCHAR AS (VALUE:c2::VARCHAR)
)
LOCATION = @aws_s3_stage
FILE_FORMAT = ingest_data;
```

### 19.3 Partitioning & Auto-Refresh

- **`PARTITION BY`** — speeds up queries by organizing files into folder directories (e.g., by country or date).
- **`AUTO_REBUILD = TRUE`** — Snowflake auto-refreshes the external table's file metadata when new files land.
- **`PATTERN`** — filters which files within a stage directory get mapped into the external table.

**Putting it together:**

```sql
CREATE OR REPLACE EXTERNAL TABLE ext_sales_partitioned (
  country      VARCHAR AS (VALUE:c1::VARCHAR),
  sales_date   DATE    AS (VALUE:c2::DATE),
  amount       NUMBER  AS (VALUE:c3::NUMBER)
)
PARTITION BY (country, sales_date)   -- organizes/prunes by these folder-derived keys
LOCATION = @aws_s3_stage/sales/
AUTO_REBUILD = TRUE                  -- keep partition metadata in sync as new files land
PATTERN = '.*sales_[0-9]{4}\.csv'    -- only map files matching this naming pattern
FILE_FORMAT = (TYPE = CSV);
```

---

## 20. Caching in Snowflake

Snowflake has **three distinct caching layers**, each serving a different performance purpose.

| Layer | Location | Duration | Requires Warehouse? | Query Profile Signal |
|---|---|---|---|---|
| **1. Query Result Cache** | Cloud Services layer | 24 hours (resets up to 31 days if re-queried) | **No** — 0 credits, milliseconds | "Query Result Reuse" |
| **2. Virtual Warehouse / Local Disk Cache** | Compute layer (local SSD on VMs) | Until warehouse is **suspended** | Yes (same warehouse) | "Percentage scanned from local disk" |
| **3. Metadata Cache** | Cloud Services layer | N/A (structural metadata) | No | Instant resolution for `COUNT(*)`, `MIN`/`MAX` |

### 20.1 Details

1. **Query Result Cache** — stores the *exact output* of a query. Accessible by **any user on any warehouse** in the account (not warehouse-specific!). Invalidated the instant the query string changes by even one character, or if the underlying table data changes.
2. **Virtual Warehouse Cache** — compute nodes have local SSDs that cache raw micro-partitions accessed during query execution. A second query on the *same warehouse* reads from this fast local disk instead of re-scanning cold cloud storage. **Invalidated the moment the warehouse suspends.**
3. **Metadata Cache** — the Cloud Services layer already knows row counts, file sizes, and min/max key values for every micro-partition, so simple aggregate queries resolve instantly without touching any warehouse at all.

### 20.2 Disabling Result Cache (for Benchmarking)

```sql
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

> 🗃️ **Analogy — Three Levels of "Already Knowing the Answer"**
>
> Think of these three caches as three different ways you avoid re-doing work. The **Query Result Cache** is like remembering the exact final answer to a question you were asked yesterday — no need to redo the whole calculation. The **Warehouse Local Disk Cache** is like keeping your recently-used reference books open on your desk (SSD) instead of walking to the library (cloud storage) again — you still have to redo *some* work, but the source material is close at hand. The **Metadata Cache** is like already knowing the total page count of a book without opening it — some facts (counts, min/max) don't require reading the content at all.

---

# Part D — Data Protection, Views & Security

## 21. Time Travel & Fail-Safe

### 21.1 Underlying Mechanism — Immutable Micro-Partitions

Data loaded into Snowflake is broken into **immutable** micro-partitions (50–500 MB). When you run `UPDATE`/`DELETE`, Snowflake **never** modifies an existing micro-partition — it writes a *new* one with the changes and keeps the old one around. How long the old versions are kept is governed by the **Data Retention Period**.

### 21.2 Retention Periods by Edition & Table Type

| Table Type | Standard Edition | Enterprise+ Edition |
|---|---|---|
| Permanent | 1 day (fixed) | 0–90 days (configurable) |
| Transient / Temporary | 0–1 day max | 0–1 day max (regardless of edition) |

Retention is inherited from the parent schema/database unless explicitly overridden:

```sql
ALTER SCHEMA raw_layer SET DATA_RETENTION_TIME_IN_DAYS = 15;
ALTER TABLE AWS_customer_load SET DATA_RETENTION_TIME_IN_DAYS = 10; -- table-level override
```

### 21.3 UNDROP — Instant Recovery

```sql
UNDROP TABLE customers;
UNDROP SCHEMA raw_layer;
```
Instantly restores a dropped object within its retention window — no backups needed.

### 21.4 Three Ways to Query Historical Data

```sql
-- 1. Offset (relative time — seconds ago)
SELECT * FROM AWS_customer_load AT(OFFSET => -200);

-- 2. Before a specific (catastrophic) Query ID
SELECT * FROM AWS_customer_load BEFORE(STATEMENT => 'query_id');

-- 3. Absolute timestamp
SELECT * FROM AWS_customer_load AT(TIMESTAMP => '2025-06-29 11:15:00'::timestamp_tz);
```

> ⏰ **Timezone Gotcha:** Timestamps used in `AT(TIMESTAMP => ...)` are interpreted using the **session's local time zone**, which **defaults to `US/Pacific`** unless you've changed it. If your team is in a different timezone and forgets this, "restoring data as of 11:15 AM" can silently pull data from a completely different real-world moment than intended. Check/set it explicitly if needed: `ALTER SESSION SET TIMEZONE = 'Asia/Kolkata';`

### 21.5 The Classic "Recreated Table" Interview Trap

**Scenario:** A table with 100 columns and 1M rows gets accidentally overwritten by `CREATE OR REPLACE TABLE ... (only 10 columns)`. Trying `AT(OFFSET => ...)` afterward **fails** — "requested snapshot is beyond the table's lifetime."

**Why:** `CREATE OR REPLACE` creates a **brand-new table object** with a new internal ID. Time Travel cannot look back past the birth of this new object — it has no memory of the old object at all.

**Solution:**
```sql
-- 1. Rename the newly (incorrectly) created table out of the way
ALTER TABLE customers RENAME TO old_customers;

-- 2. UNDROP restores the ORIGINAL table (still tracked in metadata as "dropped")
UNDROP TABLE customers;
```
This works because `CREATE OR REPLACE` internally **drops** the old table object before creating the new one — so the original 1M-row, 100-column table is recoverable via `UNDROP`, not `AT`/`BEFORE`.

### 21.6 Fail-Safe

Once the Time Travel window expires, old micro-partitions move into the **Fail-Safe Zone** for a **fixed, non-configurable 7 days**. Only **Snowflake Support** can recover data here — it's a last-resort disaster-recovery mechanism, not something end users can self-serve. Fail-Safe applies **only to Permanent tables** (Transient/Temporary get 0 days).

```sql
-- Inspect storage breakdown (active, time travel, fail-safe bytes)
SELECT * FROM snowflake.account_usage.table_storage_metrics WHERE table_catalog = 'ABC';
```

---

## 22. Zero-Copy Cloning

### 22.1 Metadata-Only Architecture

In traditional databases, cloning a 1TB table means physically duplicating 1TB of storage. In Snowflake, cloning happens **entirely in the Cloud Services layer**: a new metadata object is created that simply **points to the same immutable micro-partitions** as the source. No physical copy occurs — cloning is **instant** and costs **zero additional storage** at creation time. No warehouse is even needed.

### 22.2 When Do Costs Start?

Storage charges begin only when a **DML operation** (INSERT/UPDATE/DELETE) touches either the clone or the parent — because micro-partitions are immutable, any change writes a **new** partition. From that point, the clone is billed only for its own unique new partitions; unchanged, shared partitions remain free.

### 22.3 Tracing Clone Relationships

- Right after cloning, `ACTIVE_BYTES` for the clone shows `0` in `table_storage_metrics`.
- Every table has a unique `ID` and a `CLONE_GROUP_ID`. If they're identical, the table was never cloned. If the clone's `CLONE_GROUP_ID` matches the parent's `ID`, that's how you programmatically trace parent-child clone relationships.

### 22.4 What Can Be Cloned

```sql
CREATE DATABASE db_clone CLONE db;
CREATE SCHEMA schema_clone CLONE schema;
CREATE TABLE tbl_clone CLONE tbl;
```

Cloning a database/schema clones all standard tables, views, sequences, and custom roles — but **NOT** internal stages (user/table/named) or external tables.

### 22.5 Clone + Swap = Instant Rollback

```sql
-- Clone the table as it looked 10 minutes ago
CREATE TABLE employee_old CLONE employee AT(OFFSET => -600);

-- Swap names instantly (metadata-only operation)
ALTER TABLE employee SWAP WITH employee_old;
```
After the swap, the corrupt table (now renamed `employee_old`) can be safely dropped — this combines Time Travel + Zero-Copy Clone for a near-instant, safe rollback pattern.

> 📸 **Analogy — A Photocopy That's Actually a Bookmark**
>
> Imagine you could "photocopy" an entire 1,000-page book instantly, for free, by simply creating a bookmark that says "this book looks exactly like that other book." Only when you actually start writing notes in the margins of your "copy" does it start needing its own physical pages — and only for the pages you actually wrote on. Every unmarked page is still shared with the original. That's Zero-Copy Cloning.

---

## 23. Views — Standard, Materialized & Secure

### 23.1 Standard (Normal) Views

- Just a **named, saved query** — stores **no data**.
- Compiled and executed against base tables at query time.
- Zero storage cost.
- Used to simplify complex joins and implement row/column-level security.

**Row-level security example:**
```sql
CREATE VIEW vw_restricted AS
SELECT * FROM customer WHERE nation_key IN (13, 15);
```

**Column-level security via `EXCLUDE`:**
```sql
CREATE VIEW vwc_customer AS
SELECT * EXCLUDE (acctbal, phone) FROM snowflake_sample_data.tpch_sf1.customer;
```

### 23.2 Materialized Views

- **Physically stores pre-computed results** in database storage — extra storage cost, but extremely fast reads.
- Best when base tables don't change often but you query the aggregation frequently.
- Snowflake **automatically maintains** materialized views in the background (serverless) whenever the base table changes.

```sql
CREATE MATERIALIZED VIEW mv_daily_order_totals AS
SELECT order_date, SUM(amount) AS total_amount
FROM orders
GROUP BY order_date;
```

> ⚠️ **Critical Limitation:** A materialized view **cannot reference more than one table** — **no JOINs allowed**. It's strictly for single-table filters/aggregations. Attempting a JOIN throws: *"invalid materialized view definition: More than one table cannot be referenced."* Standard views have no such restriction.

### 23.3 Secure Views

```sql
CREATE SECURE VIEW svw_customer AS SELECT * FROM customer;
```
- By default, a view's *definition* is visible to any role with sufficient privileges via `SHOW VIEWS` or `GET_DDL` — and non-secure views can leak underlying table/column names via Query History/Query Profile.
- **Secure Views** hide the underlying query definition and base-table metadata from unauthorized users.
- **Mandatory prerequisite for Data Sharing** — you cannot share a non-secure view externally.
- Trade-off: secure views can run **slightly slower**, because certain optimizations (like filter pushdown) are intentionally disabled to prevent users from inferring hidden row ranges.

---

## 24. Streams — Change Data Capture (CDC)

### 24.1 What is a Stream?

A **Stream** is a lightweight object on a table/view that records row-level DML changes (`INSERT`/`UPDATE`/`DELETE`). Critically, a stream **does not duplicate any table data** — it's a **metadata bookmark** tracking the transactional offset of the base table, consuming negligible storage.

### 24.2 The Three Stream Metadata Columns

When you query a stream, you get all base-table columns plus:

| Column | Meaning |
|---|---|
| `METADATA$ACTION` | `INSERT` or `DELETE` |
| `METADATA$ISUPDATE` | Boolean — was this part of an `UPDATE`? |
| `METADATA$ROW_ID` | Persistent row identifier for tracking changes over time |

An `UPDATE` is represented internally as a **pair of rows**: a `DELETE` (old value) + an `INSERT` (new value), both sharing the same `METADATA$ROW_ID`, with `METADATA$ISUPDATE = TRUE`.

### 24.3 Stream Types

```sql
-- Standard (default): tracks INSERT, UPDATE, DELETE
CREATE STREAM str_default ON TABLE employee_data;

-- Append-only: tracks only INSERTs, ignores updates/deletes
CREATE STREAM str_emp ON TABLE employee_data APPEND_ONLY = TRUE;
```
There's also an **Insert-Only** stream type specifically for **External Tables / Iceberg Tables** (read-only from Snowflake's perspective) — it tracks only newly added files.

### 24.4 The Bookmark Analogy — Consuming a Stream

A stream behaves like a **bookmark in a book**. A `SELECT * FROM my_stream` query alone does **not** advance the bookmark. Only a **DML transaction that reads from the stream and writes elsewhere** (e.g., `INSERT INTO target SELECT * FROM my_stream`) actually **consumes** the stream — the moment that transaction commits, the bookmark advances and the stream's visible contents reset to empty.

> 🔖 **Analogy — A Bookmark, Not a Photocopy**
>
> If you just peek at where your bookmark sits in a novel (read-only `SELECT`), the bookmark doesn't move. Only when you actually *read past that point and physically move the bookmark forward* (a committed DML transaction that consumes the stream) does your "unread" section shrink. Peeking is free and repeatable; consuming is a one-way commitment.

### 24.5 Staleness

By default, a stream becomes **stale** (unusable, history unrecoverable) after **14 days** of inactivity. Extend it (up to 90 days) via the base table:

```sql
ALTER TABLE employee_data SET MAX_DATA_EXTENSION_TIME_IN_DAYS = 20;
```

---

## 25. Tasks — Automating Workflows

### 25.1 What is a Task?

A database object that schedules and executes a single SQL statement, a scripting block, or a **stored procedure** call — commonly used for periodic ETL/ELT jobs, cleanup scripts, and pipeline orchestration.

### 25.2 User-Managed vs. Serverless Tasks

| Type | Compute | Size Range |
|---|---|---|
| **User-Managed** | You specify a warehouse (`WAREHOUSE = compute_warehouse`) | XS – 6XL |
| **Serverless** | Snowflake auto-provisions compute in the background; you omit the warehouse | Up to 2XL |

### 25.3 Scheduling — Minutes or Cron

```sql
-- Fixed interval (max 11,520 minutes = exactly 8 days)
CREATE TASK first_task SCHEDULE = '1 MINUTE' AS SELECT 1;

-- Cron expression with timezone
CREATE TASK cron_task
  SCHEDULE = 'USING CRON 30 9 * * 1 America/Los_Angeles'
  AS INSERT INTO log_table VALUES(CURRENT_TIMESTAMP);
```

### 25.4 Tasks Are Born Suspended

Every new task starts in a **`suspended`** state and will never run until explicitly resumed:

```sql
ALTER TASK first_task RESUME;
```

### 25.5 Task Graphs (DAGs)

```sql
CREATE TASK child_task AFTER parent_task
  AS INSERT INTO child_table SELECT * FROM parent_table;
```

- A child task fires **only after its parent finishes successfully**.
- Max **1,000 tasks** per graph; max **100 parents** and **100 children** per single task.
- **Critical rule:** to alter a child task (or resume any task in a chain), the **root/parent task must be suspended**. Trying to alter a child while the parent is actively running/resumed throws an error.
- **Resuming order:** always resume tasks **bottom-up** — children first, then the root task last.

### 25.6 Conditional Execution — Only Run If a Stream Has Data

```sql
CREATE TASK process_stream_task
  SCHEDULE = '1 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('my_stream')
  AS INSERT INTO target_table SELECT * FROM my_stream;
```
If the stream is empty, the task skips entirely — no warehouse spins up, no credits burned.

---

## 26. Role-Based Access Control (RBAC)

### 26.1 The Core Principle

**Privileges are never granted directly to users.** Privileges (e.g., `SELECT` on a table, `USAGE` on a warehouse) are granted to **roles**; users are then granted roles, and inherit whatever privileges those roles carry. This decouples security management from individual user management.

> 📎 **Note — RBAC vs. DAC:** Snowflake's security model is primarily **Role-Based Access Control (RBAC)**, but it coexists with an element of **Discretionary Access Control (DAC)** — every securable object has an *owner* (by default, the role that created it), and that owner can independently grant privileges on the object to other roles at their own discretion, on top of the broader RBAC hierarchy.

> 🎫 **Analogy — Access Badges, Not Name Tags**
>
> RBAC is like an office issuing colored access badges ("Finance," "Engineering," "Visitor") rather than programming the door locks to recognize each individual employee's face. When someone joins Finance, you just hand them the Finance badge — you don't reprogram every door in the building. If a policy changes (e.g., Finance now also needs access to Room 5B), you update the *badge's* permissions once, and everyone holding that badge is automatically updated.

### 26.2 The Six System-Defined Roles

| Role | Responsibility |
|---|---|
| **`ORGADMIN`** | Highest, organization-wide tier — billing, usage, creating/deleting entire Snowflake accounts |
| **`ACCOUNTADMIN`** | Supreme admin within one account; encompasses `SYSADMIN` + `SECURITYADMIN`; access should be very restricted |
| **`SECURITYADMIN`** | Manages account-level security, grants, object privileges, users, and roles; inherits from `USERADMIN` |
| **`USERADMIN`** | Restricted to creating/managing users and roles |
| **`SYSADMIN`** | Creates/manages databases, schemas, tables, views, and warehouses; custom roles should roll up to this |
| **`PUBLIC`** | Pseudo-role auto-assigned to everyone; used for account-wide default privileges |

### 26.3 Building a Custom Role — Full Example

```sql
-- 1. Create the role
CREATE ROLE bi_role;

-- 2. Grant the role to a user
GRANT ROLE bi_role TO USER katrina;

-- 3. Grant database + warehouse usage
GRANT USAGE ON DATABASE youtube_learning TO ROLE bi_role;

-- 4. Grant warehouse usage — without this, the role has no compute to actually run queries
GRANT USAGE ON WAREHOUSE compute_wh TO ROLE bi_role;

-- 5. Grant schema usage and table select
GRANT USAGE ON SCHEMA youtube_learning.raw_layer TO ROLE bi_role;
GRANT SELECT ON ALL TABLES IN SCHEMA youtube_learning.raw_layer TO ROLE bi_role;

-- 6. Extend SELECT automatically to any FUTURE tables created in this schema
GRANT SELECT ON FUTURE TABLES IN SCHEMA youtube_learning.raw_layer TO ROLE bi_role;
```

> ⚠️ **Common Gotcha:** Steps 3 and 5 (`USAGE` on the database/schema) and step 4 (`USAGE` on the warehouse) are all independently required. Granting `SELECT` on tables alone is not enough — without `USAGE` on the parent database/schema, the role can't even "see" its way down to the table; without `USAGE` on a warehouse, the role has no compute to actually execute the query.

---

## 27. Dynamic Data Masking (DDM)

### 27.1 What is DDM?

A **column-level security feature** that masks sensitive values (credit cards, SSNs, phone numbers) at **query time**, based on the querying user's active role — while the underlying raw data on disk remains fully unmasked.

### 27.2 Creating a Masking Policy

```sql
CREATE OR REPLACE MASKING POLICY pi_masking AS (val string) RETURNS string ->
  CASE
    WHEN CURRENT_ROLE() = 'ACCOUNTADMIN' THEN val
    WHEN CURRENT_ROLE() = 'CCP' THEN REGEXP_REPLACE(val, '^([0-9]{3})', '###') -- partial mask
    WHEN CURRENT_ROLE() = 'SUPPLIER' THEN '***' -- complete mask
    ELSE 'YOU CANNOT SEE ME'
  END;
```

> ⚠️ **Rule:** the masking policy's input argument type and return type **must match exactly**, or you get a conversion error.

### 27.3 Applying / Removing a Policy

```sql
ALTER TABLE credit_card_customer MODIFY COLUMN credit_card SET MASKING POLICY pi_masking;
ALTER TABLE credit_card_customer MODIFY COLUMN credit_card UNSET MASKING POLICY;
```

### 27.4 The Interview Secret — Filters Can't Bypass Masking 🔑

If an unauthorized user filters directly on the masked column (`WHERE credit_card = '1234-5678-9012'`), **Snowflake still matches the filter against the raw, unmasked value on disk** (so the correct row is returned if it exists) — but the **displayed** value in the result is still masked. This prevents brute-force "guess and check" attacks against masked columns.

---

## 28. Account Usage vs. Information Schema

| Feature | Information Schema | Account Usage (`snowflake.account_usage`) |
|---|---|---|
| **Scope** | Database-level (local, exists inside every database) | Account-level (global, lives in the shared `SNOWFLAKE` database) |
| **Latency** | 0 (real-time) | 45 minutes – 3 hours (some views up to 24 hours) |
| **Retention** | Short (7–180 days) | 365 days (1 year) |
| **Deleted Objects** | Excluded entirely | Included, flagged via `IS_DELETED = YES` |
| **Default Access** | Granted to all users | Restricted to `ACCOUNTADMIN` by default |
| **Best For** | Immediate validation right after creating objects | Long-term usage analytics, billing audits, historical dashboards |

Information Schema also exposes specialized **table functions** for session/task histories, e.g. `INFORMATION_SCHEMA.TASK_HISTORY`.

---

## 29. Flattening Semi-Structured Data (JSON & XML)

### 29.1 The `FLATTEN` Table Function

`FLATTEN` explodes a variant object/array column into separate rows. It's joined against the base table using the **`LATERAL`** modifier (which behaves like an inline correlated subquery/loop):

```sql
SELECT * FROM JSON_table, LATERAL FLATTEN(input => data:users);
```

`FLATTEN` always produces six standard columns: `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, `THIS`. **`VALUE`** holds the actual exploded element — all further extraction happens against `VALUE`.

### 29.2 Extraction & Type-Casting Rules

- Extract nested keys with **colon notation**: `value:first_name`.
- By default, an extracted value is returned as a **variant** (wrapped in quotes).
- Use **double-colon casting** to get a proper SQL type: `value:first_name::string`, `value:user_id::int`.
- **JSON keys are strictly case-sensitive.** Extracting `value:userID` when the actual key is `userId` silently returns `NULL` — no error, just missing data. This is a very common source of silent bugs.

### 29.3 XML Extraction via `XMLGET`

- **Tag/Attribute names** — extracted with `@` (e.g., `column_name@tag`).
- **Values** — extracted with `$` (e.g., `column_name$`).
- Deeply nested XML requires **chaining multiple `LATERAL FLATTEN`** joins in sequence (e.g., flatten departments → nest into employees → nest into address tags).

---

## 30. Secure Data Sharing

### 30.1 No Data Copying — Metadata Pointers Only

Traditional data sharing = ETL export → FTP transfer → import at the consumer end. Snowflake's Secure Data Sharing involves **zero data movement** — sharing is a pure metadata pointer in the Cloud Services layer, and the consumer queries the provider's live micro-partitions in real time. Shared databases are **strictly read-only** on the consumer side.

### 30.2 Cost Allocation

| Cost | Who Pays |
|---|---|
| **Storage** | **Provider** — consumer pays $0 for storage |
| **Compute** | **Consumer** — uses their own warehouse and credits to query |

### 30.3 Direct Shares vs. Reader Accounts

| Mode | When Used | Cost Model |
|---|---|---|
| **Direct Share** | Consumer already has a Snowflake account in the same cloud/region | Standard split above |
| **Reader Account** | Consumer has **no** Snowflake account | Provider pays **both storage AND compute** — must set strict resource monitors to avoid runaway costs |

### 30.4 Security Rules

- Shared views **must be Secure Views** — standard views expose underlying base-table structure and cannot be shared.
- Shared data updates are **instant** — any write to the provider's base table is immediately visible to consumer queries.

---

# Part E — Cost Governance, Advanced Loading & Modern Pipelines

## 31. Resource Monitors

### 31.1 Purpose

**Resource Monitors** track and control credit consumption at the **Account level** or **Warehouse level**, protecting against runaway costs.

- **Account-level monitor:** covers the entire account (except serverless features), and doesn't override individual warehouse-level monitors.
- **Warehouse-level monitor:** scoped to one or more specific warehouses.

### 31.2 Configuration

- Set a **Credit Quota** (e.g., 100 credits) for a defined **interval** — Daily, Weekly, Monthly, Yearly, or Never.
- Up to **five trigger thresholds** can be configured (e.g., 50%, 75%, 90%).

| Action | Behavior |
|---|---|
| **Suspend** | Warehouse suspends after the threshold, but currently running queries are allowed to finish |
| **Suspend Immediate** | Warehouse suspends instantly, **aborting** any active queries mid-run |

**Serverless exclusion:** account-level resource monitors do **not** cover serverless features like Snowpipe, materialized view maintenance, or automatic clustering — these are billed and monitored separately.

```sql
CREATE OR REPLACE RESOURCE MONITOR ABC_MONITOR
  WITH CREDIT_QUOTA = 100
  FREQUENCY = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 50 PERCENT DO NOTIFY
    ON 90 PERCENT DO SUSPEND
    ON 99 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE COMPUTE_WH SET RESOURCE_MONITOR = ABC_MONITOR;
```

---

## 32. Micro-Partitioning & Clustering *(Unavailable — Members-Only)*

> This video is a "Members Only" video in the original playlist and its transcript was not accessible. Per the source material, it covers the core storage mechanics of Snowflake — how data is automatically sorted and stored in immutable micro-partitions, and how custom **Clustering Keys** can be used to optimize pruning for very large tables. (Micro-partition immutability itself is already covered in Section 21, in the context of Time Travel.)

---

## 33. INFER_SCHEMA — Auto-Detect Table Structure

### 33.1 The Problem It Solves

Manually writing `CREATE TABLE` statements for files with 100+ columns is tedious and error-prone. `INFER_SCHEMA` inspects staged files and automatically returns their column names, types, and nullability.

- **Supported formats:** Parquet, JSON, Avro, CSV — **not** XML (as of the time of recording).
- **Requires** files sitting in a **named** internal or external stage (does **not** work on default table stages).

### 33.2 Key Parameters

| Parameter | Purpose |
|---|---|
| `ignore_case` | `TRUE` forces detected column names to uppercase/case-insensitive; `FALSE` preserves original casing (e.g., camelCase) |
| `max_file_count` / `max_records_per_file` | Limit how many files/records are inspected, for performance |

### 33.3 Full Workflow — Detect, Create, Load

```sql
-- 1. Create the file format
CREATE OR REPLACE FILE FORMAT my_parquet_format TYPE = 'PARQUET';

-- 2. Inspect the schema
SELECT * FROM TABLE(
  INFER_SCHEMA(
    LOCATION => '@my_external_stage/data/',
    FILE_FORMAT => 'my_parquet_format',
    IGNORE_CASE => TRUE
  )
);

-- 3. Auto-create a table matching that schema
CREATE OR REPLACE TABLE user_data
  USING TEMPLATE (
    SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
    FROM TABLE(
      INFER_SCHEMA(
        LOCATION => '@my_external_stage/data/',
        FILE_FORMAT => 'my_parquet_format'
      )
    )
  );

-- 4. Load the data, matching columns by name
COPY INTO user_data
  FROM @my_external_stage/data/
  FILE_FORMAT = 'my_parquet_format'
  MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```

---

## 34. Loading Data from Azure Blob Storage

### 34.1 Secure Handshake — Azure AD Service Principal

Instead of passing raw storage keys or SAS tokens, Azure integration uses an **Azure Active Directory Service Principal**, auto-generated when you create the Storage Integration.

```sql
-- 1. Create Storage Integration
CREATE OR REPLACE STORAGE INTEGRATION azure_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'AZURE'
  ENABLED = TRUE
  AZURE_TENANT_ID = 'your-tenant-id'
  STORAGE_ALLOWED_LOCATIONS = ('azure://mystorage.blob.core.windows.net/sourcecontainer/');

-- 2. Retrieve the Snowflake App's consent info
DESCRIBE INTEGRATION azure_integration;

-- 3. Create the External Stage
CREATE OR REPLACE STAGE azure_stage
  URL = 'azure://mystorage.blob.core.windows.net/sourcecontainer/'
  STORAGE_INTEGRATION = azure_integration
  FILE_FORMAT = my_csv_format;

-- 4. Verify and load
LIST @azure_stage;
COPY INTO address FROM @azure_stage FILES = ('address_data.csv');
```

**Azure-side step:** grant the Snowflake App the **Storage Blob Data Contributor** role in Azure IAM (at the Storage Account or Container level) — this grants read/write/delete access.

---

## 35. SCD Type 2 Using Streams & Tasks

### 35.1 Pipeline Design

A classic Slowly Changing Dimension (SCD) Type 2 pipeline uses three layers:

1. **Staging Table** — holds raw uploaded data.
2. **Type 1 Table (Upsert Layer)** — holds only the *current* state, overwriting on matching business keys.
3. **Type 2 Table (History Layer)** — the true target, holding both current and all historical versions, tracked with `START_DATE`, `END_DATE`, and `IS_ACTIVE`.

### 35.2 The Task DAG

| Task | Role |
|---|---|
| Task 1 (Root) | Truncate the staging table |
| Task 2 (Child) | `COPY INTO` staging from cloud storage |
| Task 3 (Child) | Merge staging → Type 1 (upsert current state) |
| Task 4 (Child) | If the stream on Type 1 has data, run a multi-statement merge into Type 2: expire old rows (`IS_ACTIVE = FALSE`, `END_DATE = CURRENT_DATE()`) and insert new active versions |

**Important operational rule:** resume tasks **bottom-up** — children before the root.

### 35.3 The Core SCD Type 2 Merge

```sql
-- Stream on the Type 1 table
CREATE OR REPLACE STREAM str_employee ON TABLE employee_t1;

-- Merge CDC changes from the stream into the Type 2 (history) table
MERGE INTO dim_employee d
USING (
  SELECT * FROM str_employee
) s
ON d.employee_id = s.employee_id AND d.is_active = TRUE
WHEN MATCHED AND s.metadata$action = 'DELETE' AND s.metadata$isupdate = TRUE THEN
  UPDATE SET d.is_active = FALSE, d.end_date = CURRENT_DATE()
WHEN NOT MATCHED AND s.metadata$action = 'INSERT' THEN
  INSERT (employee_id, name, email, address, start_date, end_date, is_active)
  VALUES (s.employee_id, s.name, s.email, s.address, CURRENT_DATE(), '9999-12-31', TRUE);
```

This exploits the stream's paired DELETE+INSERT representation of updates (see Section 24.2): the `DELETE` half of an update expires the old historical row, and the `INSERT` half (whether a genuine new record or the "new" half of an update) creates the fresh active row.

---

## 36. Task Monitoring via AWS SNS

Applying the same alerting pattern from Section 17 (Snowpipe alerts), but for **Tasks**:

```sql
-- Notification Integration
CREATE OR REPLACE NOTIFICATION INTEGRATION task_sns_int
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = 'AWS_SNS'
  ENABLED = TRUE
  DIRECTION = OUTBOUND
  AWS_SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789012:snowflake-task-alert'
  AWS_SNS_ROLE_ARN = 'arn:aws:sns:us-east-1:123456789012:role/snowflake-monitor-role';

DESCRIBE INTEGRATION task_sns_int; -- fetch external ID/trust info

-- Attach to a task
CREATE OR REPLACE TASK load_data_task
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = '1 MINUTE'
  ERROR_INTEGRATION = task_sns_int
  AS
  COPY INTO country FROM @source_stage;
```

**`ERROR_INTEGRATION`** fires on failure (high priority for production); **`SUCCESS_INTEGRATION`** fires on every successful run (useful for testing, but normally disabled in production to avoid inbox flooding).

---

## 37. The CHANGES Clause vs. Streams

### 37.1 What is the CHANGES Clause?

A way to query historical transaction/change metadata **directly from a table or view** over a specific time window, **without creating a dedicated Stream object at all**.

### 37.2 When to Use Which

| Use Streams When... | Use CHANGES When... |
|---|---|
| You need automated, continuous, real-time ingestion via Tasks | You need ad-hoc audits or historical analysis |
| A single consumer/offset bookmark model works fine | **Multiple** team members need independent access to change history without interfering with each other's offsets |
| Reading in a DML transaction is expected to consume/advance the offset | You want to query any historical window (T1→T2) repeatedly, non-destructively |

### 37.3 Enabling & Using CHANGES

```sql
-- Prerequisite: enable change tracking (unless a Stream already exists on the table)
ALTER TABLE customer SET CHANGE_TRACKING = TRUE;

-- Default changes since a timestamp
SELECT * FROM customer
  CHANGES(INFORMATION => DEFAULT)
  AT(TIMESTAMP => $start_time);

-- Append-only (inserts only) changes within a range
SELECT * FROM customer
  CHANGES(INFORMATION => APPEND_ONLY)
  BETWEEN(TIMESTAMP => $start_time) AND(TIMESTAMP => $end_time);
```

---

## 38. Snowflake + GitHub Integration

### 38.1 How It Works

Snowflake internally clones your GitHub repo as a **Git Repository Stage** object — a live, syncable mirror containing all files, branches, tags, and commit history.

### 38.2 Setup Steps

1. **GitHub Personal Access Token (PAT)** — a classic dev token for authentication.
2. **Snowflake Secret** — securely stores your GitHub username + PAT.
3. **API Integration** — with `API_PROVIDER = git_https_api`, scoped to your repo's origin domain.
4. **Git Repository object** — links the origin URL, API integration, and secret.
5. **Fetch** — `ALTER GIT REPOSITORY ... FETCH` pulls the latest commits.
6. **Execute** — run staged SQL scripts directly from the Git stage.

```sql
-- Secret for credentials
CREATE OR REPLACE SECRET git_secret
  TYPE = PASSWORD
  USERNAME = 'your-github-username'
  PASSWORD = 'your-personal-access-token';

-- API Integration
CREATE OR REPLACE API INTEGRATION git_api_int
  API_PROVIDER = 'git_https_api'
  API_ALLOWED_PREFIXES = ('https://github.com/your-username/')
  ENABLED = TRUE;

-- Git Repository object
CREATE OR REPLACE GIT REPOSITORY my_git_repo
  API_INTEGRATION = git_api_int
  CREDENTIALS = git_secret
  ORIGIN = 'https://github.com/your-username/snowflake-scripts.git';

-- Sync latest commits
ALTER GIT REPOSITORY my_git_repo FETCH;

-- Run a SQL script straight from the main branch
EXECUTE IMMEDIATE FROM @my_git_repo/branches/main/setup_tables.sql;
```

This enables Git-backed DevOps workflows for database change management (version-controlled DDL/DML scripts).

---

## 39. Dynamic Tables

### 39.1 Concept — Declarative, Self-Refreshing Pipelines

Instead of manually orchestrating Streams + Tasks + Merge statements, a **Dynamic Table** lets you just write a `SELECT` query and declare a **Target Lag** (a freshness guarantee) — Snowflake handles the refresh scheduling and dependency ordering automatically.

### 39.2 Target Lag Options

| Option | Behavior |
|---|---|
| Time-based (e.g., `1 MINUTE`, `1 HOUR`) | Data is never older than this duration compared to base tables |
| `DOWNSTREAM` | Refreshes only when a dependent downstream table triggers a refresh |

### 39.3 Refresh Modes

- **Incremental (default/recommended)** — computes only the changed delta since last refresh; highly cost-effective.
- **Full** — recomputes the entire table from scratch every cycle.

### 39.4 Initialization Options

- **`ON_CREATE`** — populates data immediately upon creation.
- **`ON_SCHEDULE`** — stays empty until the first scheduled lag interval fires.

### 39.5 Automatic DAG Management

When multiple Dynamic Tables depend on each other, Snowflake automatically builds and maintains a **Directed Acyclic Graph (DAG)** to refresh them in the correct dependency order — you don't manually chain tasks.

```sql
CREATE OR REPLACE DYNAMIC TABLE daily_sales_summary
  TARGET_LAG = '1 MINUTE'
  WAREHOUSE = COMPUTE_WH
  REFRESH_MODE = AUTO
  INITIALIZE = ON_CREATE
  AS
  SELECT
    customer_name,
    order_date,
    SUM(amount) AS total_sales
  FROM orders_base
  GROUP BY customer_name, order_date;
```

---

## 40. SCD Type 1 & 2 Using Dynamic Tables

### 40.1 A Much Simpler Alternative

Standard SCD Type 2 (Section 35) requires multiple merge statements, task DAGs, and streams. With **Dynamic Tables**, the same logic collapses into **a single declarative SQL statement** using window functions.

### 40.2 Building SCD Type 2 with `LEAD()`

- Use the **`LEAD`** window function to find each customer's *next* change timestamp, which becomes the `END_DATE` of the *current* record.
- Replace `NULL` end dates (meaning "still active") with a sentinel far-future date, e.g. `9999-12-31`.
- Flag `IS_ACTIVE` based on whether `END_DATE` equals that sentinel date.

```sql
CREATE OR REPLACE DYNAMIC TABLE customer_scd2
  TARGET_LAG = '1 MINUTE'
  WAREHOUSE = COMPUTE_WH
  AS
  WITH next_changes AS (
    SELECT
      customer_id, customer_name, city, email, phone,
      updated_at AS start_date,
      LEAD(updated_at) OVER (PARTITION BY customer_id ORDER BY updated_at) AS raw_end_date
    FROM customer_stage
  )
  SELECT
    customer_id, customer_name, city, email, phone,
    start_date,
    NVL(raw_end_date, '9999-12-31'::TIMESTAMP) AS end_date,
    CASE WHEN raw_end_date IS NULL THEN TRUE ELSE FALSE END AS is_active
  FROM next_changes;
```

### 40.3 Building SCD Type 1 on Top of SCD Type 2

Simply layer a second Dynamic Table filtering only active rows:

```sql
CREATE OR REPLACE DYNAMIC TABLE customer_scd1
  TARGET_LAG = '1 MINUTE'
  WAREHOUSE = COMPUTE_WH
  AS
  SELECT customer_id, customer_name, city, email, phone
  FROM customer_scd2
  WHERE is_active = TRUE;
```

The resulting **lineage** is: `Staging Table → SCD Type 2 Dynamic Table → SCD Type 1 Dynamic Table` — and Snowflake automatically refreshes this entire chain in the correct order whenever the staging data changes.

---

# Part F — Applications, Governance & Programmability

## 41. Streamlit in Snowflake

### 41.1 What It Enables

Build **interactive web/data apps directly inside Snowflake** — no external servers, container registries, or deployment pipelines. Everything runs natively, fully integrated with Snowflake's RBAC and compute resources.

### 41.2 Getting Started

```python
import streamlit as st
from snowflake.snowpark.context import get_active_session

session = get_active_session()
```

### 41.3 Common Visual Elements

| Element | Purpose |
|---|---|
| `st.title()`, `st.caption()` | Headings and descriptions |
| `st.columns()` | Side-by-side grid layout |
| `st.metric()` | KPI scorecards (e.g., Total Warehouses, Avg Query Time) |
| `st.bar_chart()`, `st.line_chart()` | Dynamic charts; pass `use_container_width=True` to auto-scale to the browser |

### 41.4 Writing SQL Inside a Streamlit App

```python
def run_query(query):
    return session.sql(query).to_pandas()
```
Wrap SQL (e.g., querying `ACCOUNT_USAGE` for warehouse metering or query histories) inside a Python function and convert results directly to a Pandas DataFrame for visualization.

### 41.5 Security

Streamlit apps are securable, database-level objects — they execute using the privileges of the **active role** (`ACCOUNTADMIN` or a custom role), so unauthorized users can't use the app to reach backend data they wouldn't otherwise have access to.

---

## 42. Tag-Based Masking Policies

### 42.1 The Governance Bottleneck

Manually assigning masking policies column-by-column doesn't scale — if a new PII column is added and an admin forgets to mask it, sensitive data leaks. **Tags** solve this by classifying columns with metadata labels that carry masking policies automatically.

### 42.2 What Are Tags?

Schema-level objects representing a **key-value pair** (e.g., key: `customer_sensitive`, value: `pi_varchar`) used to classify, group, and track compliance across columns, tables, schemas, or even warehouses.

### 42.3 Full Tag-Based Masking Workflow

```sql
-- Step 1: Define the tag with allowed values
CREATE OR REPLACE TAG customer_sensitive
  ALLOWED_VALUES 'pi_varchar', 'pi_number';

-- Step 2: Create masking policies matching the target data types
CREATE OR REPLACE MASKING POLICY mask_customer_pii AS (val string) RETURNS string ->
  CASE WHEN current_role() = 'ACCOUNTADMIN' THEN val ELSE '*********' END;

-- Step 3: Bind the masking policy to the tag
ALTER TAG customer_sensitive SET MASKING POLICY mask_customer_pii;

-- Step 4: Assign the tag to a column — masking is now automatic
ALTER TABLE customer_master MODIFY COLUMN email SET TAG customer_sensitive = 'pi_varchar';
```

### 42.4 Multi-Policy Tag Mapping

A **single tag** can hold multiple bound masking policies (one for strings, one for numbers, etc.). When the tag is applied to a column, Snowflake's compiler **automatically matches** the correct policy to that column's actual data type — e.g., mapping the text-masking policy to `VARCHAR` columns and the numeric one to `NUMBER`/`INT` columns.

### 42.5 Governance Dashboards

Snowsight's **Governance tab** gives admins a birds-eye view: most-used tags, masked tables, and active row-access/masking rules across the entire account.

---

## 43. Loading API Data into Snowflake

### 43.1 The Three-Layer Native Ingestion Stack

Snowflake can securely pull data from external APIs (public or key-protected) **without any third-party ETL tool**, using three native security layers:

| Layer | Role |
|---|---|
| **Network Rule** | The "gateway" — defines approved external egress destinations (domains, ports) |
| **Secret** | The "credential vault" — stores API keys/tokens securely, never hard-coded in plain text |
| **External Access Integration** | The "middleman" — binds the Network Rule + Secret together, granting specific users/functions permission to make outbound internet requests |

### 43.2 Setting Up Outbound Access

```sql
-- Network Rule: approved egress destination
CREATE OR REPLACE NETWORK RULE tmdb_api_rule
  TYPE = HOST_PORT
  MODE = EGRESS
  VALUE_LIST = ('api.themoviedb.org');

-- Secret: store the API key securely
CREATE OR REPLACE SECRET tmdb_api_key
  TYPE = GENERIC_STRING
  SECRET_STRING = 'your_private_api_key';

-- External Access Integration: bind them together
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION tmdb_integration
  ALLOWED_NETWORK_RULES = (tmdb_api_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (tmdb_api_key)
  ENABLED = TRUE;
```

### 43.3 Retrieving Data with Python UDTFs

Write a Python **User-Defined Table Function (UDTF)** using the `requests` library, `yield`-ing records row-by-row into a relational structure. To allow the function to make external calls, reference the integration explicitly:

```sql
EXTERNAL_ACCESS_INTEGRATIONS = (tmdb_integration)
SECRETS = ('cred' = tmdb_api_key)
```

### 43.4 Bulk Loading via Stored Procedures

A Python Stored Procedure can handle **paginated** API calls, gather the JSON responses, and write raw records straight into a table. Since API arrays typically arrive as raw Python strings, use **`PARSE_JSON`** when inserting them, so the data lands in a properly indexable `VARIANT`/`ARRAY` column rather than as a plain text blob:

```sql
-- Without PARSE_JSON, response_body would be stored as inert plain text
INSERT INTO movies_raw (payload)
SELECT PARSE_JSON(response_body)
FROM staging_api_responses;
```

Once stored this way, the payload becomes a genuine `VARIANT`, so you can immediately query it with the same path/flatten syntax covered in Sections 8, 29, and 44 (e.g., `payload:title::string`, `LATERAL FLATTEN(input => payload:results)`).

---

## 44. Nested JSON — Double/Hierarchical Flattening

### 44.1 The Challenge

Real-world API payloads often contain **multiple nested layers** — e.g., a `company` object containing an array of `employees`, each of which contains nested arrays of `skills` and `projects`. Standard relational joins can't handle this; you need to explode each array layer individually.

### 44.2 Variant Path Access

Individual keys inside a `VARIANT` column are accessed with case-sensitive dot/bracket/colon notation:

```
data:company::string
data:employees[0]:name::string
```

### 44.3 Chained (Recursive) LATERAL FLATTEN

For deeply nested structures, chain multiple `LATERAL FLATTEN` clauses — each child flatten takes the **output value of its parent flatten** as its own input:

```sql
SELECT
  data:company::string AS company_name,
  emp.value:name::string AS employee_name,
  skill.value::string AS skill_name,
  proj.value:project_name::string AS project_name
FROM raw_json_table,
LATERAL FLATTEN(input => data:employees) emp,
LATERAL FLATTEN(input => emp.value:skills) skill,
LATERAL FLATTEN(input => emp.value:projects) proj;
```

### 44.4 Type Casting & Avoiding Ambiguity

- Variant path values remain quoted variants until explicitly cast, e.g. `value:salary::int`, `value:is_active::boolean`.
- When using **multiple** lateral flattens, always **prefix output keys with their respective flatten alias** (`emp.value` vs. `skill.value`) — otherwise you get ambiguous column compilation errors, since multiple flattens all expose a column literally named `VALUE`.

> 🪆 **Analogy — Russian Nesting Dolls**
>
> A deeply nested JSON payload is like a set of Russian nesting dolls: `company` contains `employees`, which contains `skills` and `projects` inside *each* employee. You can't just "open" the whole set at once — you open the outer doll (`LATERAL FLATTEN` on `employees`), and *for each one you find inside*, you open it again (`LATERAL FLATTEN` on `skills`/`projects`) to get to the next layer. Each `LATERAL FLATTEN` is one "opening" step, feeding directly into the next.

---

## 45. Snowpipe on Azure — Auto-Ingest Pipeline

### 45.1 Different Event Architecture Than AWS

Unlike AWS Snowpipe (which uses SQS), **Azure Snowpipe** relies on **Azure Event Grid** + **Azure Storage Queues** to broadcast file-arrival notifications.

### 45.2 Azure-Side Setup

1. Create an Azure Storage Account + Blob Container.
2. Create an **Azure Storage Queue** to receive notifications.
3. Configure an **Event Grid Subscription** on the storage account, filtered to **"Blob Created"** events, targeting that Storage Queue.
4. **Prerequisite CLI step** — register the Event Grid provider namespace before creating subscriptions:
   ```bash
   az provider register --namespace Microsoft.EventGrid
   ```

### 45.3 Snowflake-Side Notification Integration

```sql
CREATE OR REPLACE NOTIFICATION INTEGRATION azure_notify_int
  ENABLED = TRUE
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = AZURE_STORAGE_QUEUE
  AZURE_STORAGE_QUEUE_PRIMARY_URI = 'https://mystorage.queue.core.windows.net/myqueue'
  AZURE_TENANT_ID = 'your_azure_tenant_id';
```

### 45.4 Role Delegation

Describe the Storage and Notification Integrations to retrieve the generated Active Directory App Service Principal credentials, then grant it:
- **Storage Blob Data Contributor** — for storage access.
- **Storage Queue Data Contributor** — for queue access.

### 45.5 Creating the Pipe

```sql
CREATE OR REPLACE PIPE raw_azure_pipe
  AUTO_INGEST = TRUE
  INTEGRATION = 'AZURE_NOTIFY_INT'
  AS
  COPY INTO target_table
    FROM @azure_external_stage/
    FILE_FORMAT = my_csv_format;
```

---

## 46. Clustering & Query Acceleration Service *(Unavailable — Members-Only)*

> This video is a "Members Only" video in the original playlist and its transcript was not accessible. Per the source material, it covers advanced query performance optimization — how Snowflake handles data clustering at scale, and the **Query Acceleration Service (QAS)**, illustrated with real-world workload examples.

---

## 47. Stored Procedures

### 47.1 What is a Stored Procedure?

A database-level object containing **reusable, procedural logic**. Unlike plain SQL queries or UDFs (which are strictly for calculations/return values), stored procedures can execute **administrative, multi-step tasks**: DDL/DML, truncating tables, triggering copy commands, and writing to audit tables — all in sequence.

### 47.2 Snowflake Scripting Block Structure

```sql
DECLARE
  -- variable declarations with data types
BEGIN
  -- main business/query logic
EXCEPTION
  -- capture, log, and recover from failures
END;
```

| Block | Purpose |
|---|---|
| `DECLARE` | Declare internal variables and their types |
| `BEGIN ... END` | Main execution logic |
| `EXCEPTION` | Catch and handle compilation/execution failures |

### 47.3 Variable Assignment & Referencing

```sql
-- Assign a value inside the execution block
total_bonus := base_salary * 0.10;

-- Reference a variable inside a SQL statement (prefix with colon)
INSERT INTO log VALUES (:total_bonus);
```

### 47.4 Exception Handling

```sql
EXCEPTION
  WHEN OTHER THEN
    RETURN 'FAILED: ' || SQLERRM;
```
If any statement fails, execution jumps to the `EXCEPTION` block — letting you return a descriptive error payload (or trigger a rollback) instead of letting the whole pipeline crash silently.

### 47.5 Practical Orchestration Example

A single stored procedure can automate an entire daily pipeline: truncate a staging table → run a bulk `COPY INTO` → apply validation rules → write a timestamp/success marker to an audit log — all triggered with one command:

```sql
CALL refresh_pipeline();
```

---

# Part G — Interview Preparation

## 48. Comprehensive Interview Questions & Answers

### Architecture & Fundamentals

**Q1. What are the three layers of Snowflake's architecture, and what does each one do?**
Ans: (1) **Cloud Services Layer** ("the brain") — handles authentication/access control, metadata management, query optimization, and infrastructure management. (2) **Virtual Warehouse Layer** ("compute") — independent compute clusters that query, load, and unload data; warehouses can scale without affecting each other. (3) **Data Storage Layer** ("storage") — centralized, compressed, columnar, encrypted storage organized into immutable micro-partitions.

**Q2. Is Snowflake a cloud provider like AWS or GCP?**
Ans: No. Snowflake is a SaaS platform that **runs on top of** a public cloud provider (AWS, Azure, or GCP), chosen at account creation. Snowflake is not itself infrastructure — it fully manages the underlying compute/storage on whichever cloud you selected, and customers cannot access the raw underlying cloud resources directly.

**Q3. What makes Snowflake's architecture a "hybrid" of shared-disk and shared-nothing systems?**
Ans: It's shared-disk in that there's one central, shared data repository accessible by all compute nodes. It's shared-nothing in that query processing uses MPP (Massively Parallel Processing) — each compute node works on its own local slice of data during execution — combining centralized data management with parallel-processing performance.

**Q4. Do DDL statements or simple metadata queries require an active virtual warehouse?**
Ans: No. Statements like `CREATE TABLE` and simple metadata-only queries (e.g., `SELECT COUNT(*)`) are resolved entirely by the Cloud Services layer's metadata cache, without spinning up compute — so they incur no warehouse credit cost.

### Virtual Warehouses & Pricing

**Q5. What is the difference between vertical and horizontal scaling of a virtual warehouse?**
Ans: **Vertical scaling** (scale up/down) changes the warehouse's *size* (e.g., Small → Large) to speed up a single complex query with more raw CPU/RAM. **Horizontal scaling** (scale out/in) uses **multi-cluster warehouses** to add/remove additional clusters of the *same size* to handle higher concurrency and prevent query queueing — it doesn't make any single query faster, it lets more queries run simultaneously.

**Q6. How is Snowflake's compute billed?**
Ans: Per-second billing, but with a **60-second minimum** per warehouse activation — a query finishing in 42 seconds is still billed for a full 60 seconds. Credit consumption scales with warehouse size (e.g., XS = 1 credit/hour, Small = 2, Medium = 4, up to 6XL = 512 credits/hour), multiplied by the per-credit dollar rate for your edition and region.

**Q7. Why might a larger warehouse sometimes be more cost-effective than a smaller one for the same job?**
Ans: Because credit consumption is size × time. If a Large warehouse (8 credits/hour) finishes a job 4× faster than a Small warehouse (2 credits/hour), the total credits consumed can end up roughly equal — but you save significant wall-clock time, which is a net win with no added cost.

### Data Types, Constraints & Table Types

**Q8. Does Snowflake enforce PRIMARY KEY and FOREIGN KEY constraints?**
Ans: No. These constraints can be declared (for documentation/BI-tool compatibility) but are **not enforced** during DML — duplicate primary keys can be inserted successfully. The **only** constraint Snowflake strictly enforces is **`NOT NULL`**.

**Q9. What are the four table types in Snowflake, and how do their Time Travel/Fail-Safe windows differ?**
Ans: **Permanent** (default) — 0–90 days Time Travel (Enterprise+), 7 days Fail-Safe; ideal for critical long-term data. **Transient** — max 1 day Time Travel, 0 days Fail-Safe; ideal for staging tables that can be re-ingested. **Temporary** — session-scoped, max 1 day Time Travel, 0 days Fail-Safe; ideal for session-only work. **External** — read-only pointer to data outside Snowflake, 0 days Time Travel/Fail-Safe, zero Snowflake storage cost.

**Q10. If a Temporary table and a Permanent table share the same name in a schema, which one gets used?**
Ans: The **Temporary table always takes precedence** for any operation (queries, drops) referencing that name, for as long as the session/temporary table exists. Once the temporary table is dropped or the session ends, operations automatically fall back to targeting the Permanent table.

### File Formats, Stages & Loading

**Q11. What's the difference between the four ways to load data into Snowflake?**
Ans: **Snowsight UI** — simple drag-and-drop for files up to 250MB with schema inference. **SnowSQL CLI** — `PUT` (upload to stage) + `COPY INTO` (load to table), suited for large files/automation. **Snowpipe** — continuous, event-driven, serverless auto-ingestion. **Third-party ETL/ELT or custom code** — tools like Matillion/Informatica or Python scripts for complex pipelines.

**Q12. What are `PUT` and `GET`, and why can't they run from the browser UI?**
Ans: `PUT` uploads a local file into an internal stage; `GET` downloads a file from an internal stage to local disk. They require direct filesystem access on the client machine, which a browser sandbox doesn't provide — so they can only be run via the SnowSQL CLI.

**Q13. What happens if you run `COPY INTO` twice on the exact same already-loaded file?**
Ans: By default, nothing loads the second time — Snowflake tracks ingested files in a **64-day metadata cache** and silently skips files it recognizes as already loaded. To force a reload, use `COPY INTO ... FORCE = TRUE`.

**Q14. What's the difference between `ON_ERROR = CONTINUE` and `ON_ERROR = SKIP_FILE`?**
Ans: `CONTINUE` loads all clean rows from a file and simply skips the corrupted ones. `SKIP_FILE` discards the **entire file** the moment even a single row fails to parse.

**Q15. What are User Stages, Table Stages, and Named Internal Stages, and how do they differ?**
Ans: **User Stage (`@~`)** — auto-created per user, personal-use only, cannot be dropped/altered. **Table Stage (`@%table`)** — auto-created per table, owner-only access, no transformations allowed during copy, cannot be dropped/altered. **Named Internal Stage** — explicitly created via `CREATE STAGE`, reusable and configurable, supports custom file formats and multi-user access. Only Named stages appear in `SHOW STAGES`.

### Semi-Structured Data

**Q16. What is the VARIANT data type, and why is it useful?**
Ans: A Snowflake-proprietary type holding up to 16MB of arbitrary semi-structured data (a whole JSON object, XML doc, or array) in a single column — allowing ingestion of raw semi-structured files without pre-defining a rigid schema upfront.

**Q17. What does the FLATTEN function do, and why is LATERAL needed alongside it?**
Ans: `FLATTEN` explodes a variant array/object into separate rows (with standard columns `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, `THIS`). `LATERAL` is needed to join this row-exploding table function against the base table row-by-row, behaving like an inline correlated subquery — without it, you can't correlate the flattened output back to its originating row.

**Q18. Why might `value:userID::string` silently return NULL even though the JSON clearly has that field?**
Ans: JSON key extraction in Snowflake is **case-sensitive**. If the actual key in the JSON is `userId` (lowercase `d`) and you query `userID` (uppercase `D`), Snowflake won't find a match and returns `NULL` — with no error to warn you, making this a common silent bug.

**Q19. How do you handle deeply nested JSON with multiple array layers (e.g., company → employees → skills)?**
Ans: Chain multiple `LATERAL FLATTEN` clauses, where each child flatten's `input =>` references the **value column of its parent flatten** (e.g., `LATERAL FLATTEN(input => data:employees) emp`, then `LATERAL FLATTEN(input => emp.value:skills) skill`). Always alias each flatten and prefix output columns with the correct alias to avoid ambiguous-column errors.

### Time Travel, Fail-Safe & Cloning

**Q20. What is the maximum Time Travel retention period, and does it depend on the Snowflake edition?**
Ans: For Permanent tables, Standard Edition caps retention at 1 day; Enterprise Edition and above allow 0–90 days (configurable). Transient and Temporary tables are capped at 1 day maximum regardless of edition.

**Q21. Why can't you use `AT(OFFSET => ...)` to recover a table after running `CREATE OR REPLACE TABLE` on it?**
Ans: `CREATE OR REPLACE` internally drops the old table object and creates a **brand-new object with a new internal ID**. Time Travel's `AT`/`BEFORE` clauses can only look back within the lifetime of the *current* object — they cannot see past its creation moment. The correct recovery approach is `UNDROP`, which restores the previous (dropped) table object directly.

**Q22. What is Fail-Safe, and how is it different from Time Travel?**
Ans: Fail-Safe is a **fixed, non-configurable 7-day** window that begins after a table's Time Travel retention period expires. Unlike Time Travel (which users can query directly via `AT`/`BEFORE`/`OFFSET`), Fail-Safe data can only be recovered by **Snowflake Support**, as a last-resort disaster-recovery mechanism. It applies only to Permanent tables.

**Q23. Why is Zero-Copy Cloning "free" at the moment of creation, and when do costs start accumulating?**
Ans: Cloning creates a new metadata object pointing to the same immutable micro-partitions as the source — no physical data is duplicated, so there's no storage cost and no compute/warehouse is needed. Costs begin only once a DML operation modifies either the clone or the parent, because immutable micro-partitions force any change to write a brand-new partition — from that point, only the new, unique partitions are billed.

**Q24. How can you combine Time Travel and Zero-Copy Cloning to roll back a corrupted table?**
Ans: Clone the table as it looked at a safe point in the past using `CREATE TABLE x_old CLONE x AT(OFFSET => -600)`, then instantly swap it with the current corrupted table using `ALTER TABLE x SWAP WITH x_old` — a metadata-only rename operation. The now-renamed corrupt table can then be safely dropped.

### Views

**Q25. What is the key limitation of Materialized Views in Snowflake?**
Ans: A materialized view **cannot reference more than one table** — no JOINs are allowed. It's designed strictly for single-table filters/aggregations. Standard views have no such restriction and support unlimited joins.

**Q26. Why are Secure Views required for Data Sharing?**
Ans: Standard (non-secure) views expose their underlying query definition and base-table structure (via `SHOW VIEWS`, `GET_DDL`, or Query History/Profile) — which would leak the provider's internal schema to consumer accounts. Secure Views hide this definition entirely, making them a mandatory prerequisite for sharing views externally.

### Streams & CDC

**Q27. What are the three metadata columns exposed by a Stream, and what does each represent?**
Ans: `METADATA$ACTION` (`INSERT` or `DELETE`), `METADATA$ISUPDATE` (boolean — was this row change part of an `UPDATE`?), and `METADATA$ROW_ID` (a persistent identifier used to track a specific row's changes over time).

**Q28. How does a Stream represent an UPDATE operation?**
Ans: As a **pair of rows** — a `DELETE` row (representing the old value) and an `INSERT` row (representing the new value) — both sharing the same `METADATA$ROW_ID`, with `METADATA$ISUPDATE` set to `TRUE` on both.

**Q29. Does simply running `SELECT * FROM my_stream` consume/advance the stream's offset?**
Ans: No. Only a **DML transaction** that reads from the stream and commits a write elsewhere (e.g., `INSERT INTO target SELECT * FROM my_stream`) actually advances the stream's offset and clears its visible contents. A read-only `SELECT` leaves the offset untouched, so it can be run repeatedly without side effects.

**Q30. What happens if a Stream becomes stale, and how do you prevent it?**
Ans: By default, a stream becomes stale (unusable, with its historical offset permanently lost) after **14 days** of inactivity if the base table's changes aren't consumed. You can extend this window up to 90 days via `ALTER TABLE ... SET MAX_DATA_EXTENSION_TIME_IN_DAYS = <n>`.

**Q31. When would you use the CHANGES clause instead of a Stream?**
Ans: When you need **multiple independent consumers** to query change history over arbitrary historical windows without interfering with each other (Streams maintain only a single shared offset/bookmark, so reading via one consumer's DML transaction would clear it for everyone else). CHANGES is better for ad-hoc audits and multi-team historical analysis; Streams are better for automated, continuous, single-pipeline ingestion via Tasks.

### Tasks

**Q32. What state is a newly created Task in, and what must you do before it runs?**
Ans: Every new task starts in a **`suspended`** state. It will never fire on its schedule until you explicitly run `ALTER TASK <name> RESUME;`.

**Q33. What is the rule for altering or resuming tasks that are part of a DAG (task graph)?**
Ans: To alter a child task (or resume any task in a parent-child chain), the **root/parent task must be in a suspended state** first. Additionally, when resuming a task graph, you should resume tasks **bottom-up** — children first, then the root task last.

**Q34. How can a Task avoid running (and consuming compute) when there's nothing new to process?**
Ans: Add a `WHEN` clause using `SYSTEM$STREAM_HAS_DATA('stream_name')` — the task will skip its scheduled run entirely (no warehouse spin-up, no credits) if the referenced stream is currently empty.

### Security & RBAC

**Q35. What is the core principle of Snowflake's RBAC model?**
Ans: Privileges are **never granted directly to users** — they're granted to **roles**. Users are then granted roles and inherit the privileges those roles carry. This decouples day-to-day user management from the underlying security model.

**Q36. Name the six system-defined roles in Snowflake, from highest to lowest scope.**
Ans: `ORGADMIN` (organization-wide billing/account management) → `ACCOUNTADMIN` (supreme admin within one account, includes SYSADMIN + SECURITYADMIN) → `SECURITYADMIN` (manages grants, users, roles account-wide) → `USERADMIN` (creates/manages users and roles only) → `SYSADMIN` (creates/manages databases, schemas, tables, warehouses) → `PUBLIC` (auto-assigned pseudo-role for account-wide defaults).

**Q37. What is Dynamic Data Masking, and can a user bypass it using a WHERE filter on the masked column?**
Ans: DDM masks sensitive column values at query time based on the querying user's role, while leaving the raw data unmasked on disk. A user **cannot bypass it via filtering** — Snowflake evaluates the filter condition against the true, unmasked underlying value (so a matching row is still returned if it exists), but the value displayed in the result set remains masked regardless.

**Q38. What are Tag-Based Masking Policies, and what governance problem do they solve?**
Ans: They solve the problem of manually assigning masking policies column-by-column (which is error-prone at scale — a forgotten new PII column stays exposed). A Tag is a key-value metadata label bound to a masking policy; once a tag is applied to any column, its correct masking policy is automatically inherited — and a single tag can map to multiple policies, auto-matched to each column's actual data type.

### Account Usage, Sharing & Governance

**Q39. What's the key trade-off between Information Schema and Account Usage as metadata sources?**
Ans: Information Schema offers **0-latency, real-time** data but with **short retention** (7–180 days) and database-local scope. Account Usage offers **365-day retention** and **account-wide** scope, but with **latency of 45 minutes to 3 hours** (sometimes up to 24 hours) and restricted default access (ACCOUNTADMIN only).

**Q40. In Snowflake's Secure Data Sharing, who pays for storage and who pays for compute?**
Ans: The **Provider** pays for storage (consumer pays $0 for storage). The **Consumer** pays for compute, using their own virtual warehouse to query the shared data — except in a **Reader Account** scenario (used when the consumer has no Snowflake account at all), where the Provider pays for **both** storage and compute, making strict resource monitors essential there.

**Q41. Why must a shared view be a Secure View, and can standard views be shared?**
Ans: Standard views expose their underlying query definition and base-table schema, which would leak the provider's internal data model to external consumers. Secure Views hide this information entirely, so only Secure Views are permitted in a Snowflake Share.

### Resource Monitors & Cost Control

**Q42. What's the difference between the "Suspend" and "Suspend Immediate" actions on a Resource Monitor trigger?**
Ans: **Suspend** stops the warehouse from starting new queries once the credit threshold is hit, but lets any **currently running** queries finish naturally. **Suspend Immediate** stops the warehouse instantly, **aborting** any queries that are actively running, to halt credit consumption as fast as possible.

**Q43. Do account-level Resource Monitors control credit usage from serverless features like Snowpipe?**
Ans: No. Account-level resource monitors explicitly **exclude** serverless features (Snowpipe, materialized view background maintenance, automatic clustering) — these are billed and must be monitored through separate mechanisms.

### Advanced Loading & Modern Pipelines

**Q44. What does INFER_SCHEMA do, and what's a key limitation of it?**
Ans: It inspects files already sitting in a **named** stage (not a default table stage) and automatically detects column names, data types, and nullability — eliminating manual `CREATE TABLE` authoring for wide files. Its key limitation: it supports Parquet, JSON, Avro, and CSV, but **not XML**.

**Q45. What is the fundamental difference between Streams+Tasks-based SCD Type 2 and Dynamic-Table-based SCD Type 2?**
Ans: The Streams+Tasks approach is **imperative** — you manually design a multi-step task DAG (truncate → load → upsert → conditional merge) with explicit CDC handling via a Stream. The Dynamic Table approach is **declarative** — you write a single `SELECT` query (using window functions like `LEAD()` to compute `END_DATE`/`IS_ACTIVE`) and simply set a `TARGET_LAG`; Snowflake handles all refresh scheduling and dependency management automatically.

**Q46. In the SCD Type 2 Dynamic Table pattern, what does the LEAD() window function accomplish?**
Ans: For each customer, `LEAD(updated_at) OVER (PARTITION BY customer_id ORDER BY updated_at)` looks ahead to find the timestamp of that customer's *next* change — which becomes the `END_DATE` of the *current* row. If there is no next change (the customer's latest state), `LEAD()` returns `NULL`, which is then replaced with a sentinel far-future date (e.g., `9999-12-31`) to represent "still active."

**Q47. What are the three native Snowflake objects needed to securely call an external API, and what does each do?**
Ans: A **Network Rule** (defines approved outbound domains/ports), a **Secret** (securely stores the API key/token, never hard-coded), and an **External Access Integration** (binds the Network Rule and Secret together and grants specific functions/procedures permission to actually make the outbound call).

**Q48. Why should PARSE_JSON be used when inserting API response arrays into a Snowflake table?**
Ans: Because API responses are typically parsed in Python as plain strings; without `PARSE_JSON`, they'd be stored as inert text blobs. `PARSE_JSON` converts them into genuine `VARIANT`/`ARRAY` values, which are properly indexable and queryable using Snowflake's path/flatten syntax.

**Q49. What is a Snowflake Stored Procedure, and how is it different from a UDF?**
Ans: A Stored Procedure contains reusable **procedural** logic that can perform multi-step administrative tasks (DDL/DML, truncation, orchestration, exception handling) in sequence. A UDF (User-Defined Function), by contrast, is strictly for **calculations** — it takes inputs and returns a single computed value/table, without the branching, exception-handling, or side-effecting administrative capabilities of a stored procedure.

**Q50. In Snowflake Scripting, what's the difference between `:=` and `:` when working with variables?**
Ans: `:=` is the **assignment operator**, used to assign a value to a declared variable inside the execution block (e.g., `total_bonus := base_salary * 0.10;`). A leading colon `:` is used to **reference** a variable or parameter *inside a SQL statement* (e.g., `INSERT INTO log VALUES (:total_bonus);`) — it tells the SQL engine to substitute the variable's current value at that point.

---

## 49. Conclusion

Across these 45 videos, the Snowflake platform reveals a consistent design philosophy: **separate storage from compute**, **automate everything that can be automated** (schema inference, clustering, background materialized-view refresh, dynamic table pipelines), and **charge only for what's actually used** (per-second compute billing, zero-cost cloning until data actually changes, zero storage cost for external tables and data consumers).

The core building blocks form a clear progression:
- **Storage & Compute Separation** underlies everything — Virtual Warehouses (compute) are completely decoupled from the Data Storage layer, letting you scale each independently.
- **Micro-partition immutability** is the single mechanical idea behind both **Time Travel** (old partitions are kept, not overwritten) and **Zero-Copy Cloning** (new objects just point to existing partitions until something changes).
- **Streams, Tasks, and Dynamic Tables** represent an evolution from manual CDC orchestration toward fully declarative, self-managing pipelines.
- **RBAC, Dynamic Data Masking, and Tag-Based Masking** layer security controls that scale from a handful of tables to an entire enterprise data estate without manual per-column upkeep.
- **Secure Data Sharing, External Tables, and native API/Git integrations** all reflect the same "don't copy data if you don't have to" principle applied at increasingly larger scales — across accounts, across clouds, and even across version-controlled codebases.

> 🔑 **One-Line Takeaway**
>
> Snowflake's entire architecture — from micro-partitions to Time Travel to Zero-Copy Cloning to Dynamic Tables — is built around one repeated idea: **store data once, immutably, and layer cheap metadata (pointers, bookmarks, tags, masking rules) on top of it to get performance, history, security, and automation almost for free.**
