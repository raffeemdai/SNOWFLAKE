# Snowflake — Consolidated Reference Notes

A single, organized reference combining: Internal vs External Stages, a quick simplified recap, an Iceberg Tables pointer, and 10 real-time interview/project scenarios (with full code + line-by-line explanations).

## Table of Contents

**Part A — Stages**
- [A1. What is a Stage?](#a1-what-is-a-stage)
- [A2. Internal Stage](#a2-internal-stage)
- [A3. External Stage](#a3-external-stage)
- [A4. Internal vs External — Comparison Table](#a4-internal-vs-external--comparison-table)
- [A5. Data Flow Diagrams](#a5-data-flow-diagrams)
- [A6. When to Use Which](#a6-when-to-use-which)
- [A7. Storage Integration](#a7-storage-integration)
- [A8. Simplified Recap (Plain-English + Extra Interview Q&A)](#a8-simplified-recap)
- [A9. Interview Q&A Bank (Stages)](#a9-interview-qa-bank-stages)
- [A10. Key Summary](#a10-key-summary)

**Part B — Iceberg Tables**
- [B1. Managed vs Self-Managed Iceberg Tables (reference)](#b1-managed-vs-self-managed-iceberg-tables)

**Part C — Real-Time Scenarios (Set 1)**
- [C1. Dynamic S3 Date-Folder Loads](#c1-dynamic-s3-date-folder-loads)
- [C2. Valid Data Load + Auto-Send Failures to Client](#c2-valid-data-load--auto-send-failures-to-client)
- [C3. Schema Evolution](#c3-schema-evolution)
- [C4. Dynamic Multi-Folder → Multi-Table Ingestion](#c4-dynamic-multi-folder--multi-table-ingestion)
- [C5. Dynamic Table Creation from Single-Cell CSV Headers](#c5-dynamic-table-creation-from-single-cell-csv-headers)
- [C6. Tracking DDL Changes (Schema Audit Log)](#c6-tracking-ddl-changes-schema-audit-log)
- [C7. Summary Table — Set 1](#c7-summary-table--set-1)

**Part D — Real-Time Scenarios (Set 2)**
- [D1. Load + Validate in a Single Stored Procedure](#d1-load--validate-in-a-single-stored-procedure)
- [D2. Real-Time Pipeline — Dynamic Tables + Alerts](#d2-real-time-pipeline--dynamic-tables--alerts)
- [D3. Loading Excel Files via Snowpark (Python)](#d3-loading-excel-files-via-snowpark-python)
- [D4. Auto Table Creation & Load from S3 via Snowpark + Task](#d4-auto-table-creation--load-from-s3-via-snowpark--task)

**Part E — Cross-Cutting**
- [E1. Patterns That Repeat Across All Scenarios](#e1-patterns-that-repeat-across-all-scenarios)
- [E2. Validation Notes on This Consolidation](#e2-validation-notes-on-this-consolidation)

---

# Part A — Stages

## A1. What is a Stage?

A **stage** in Snowflake is a location used to store data files for loading data *into* Snowflake tables or unloading data *out of* Snowflake tables.

```text
Source Files
     ↓
   Stage
     ↓
Snowflake Table
```

Stages are used with commands such as:

```sql
PUT
GET
COPY INTO
```

---

## A2. Internal Stage

An **Internal Stage** is a storage location fully managed by **Snowflake**. Files sit in Snowflake-managed cloud storage — Snowflake handles the underlying bucket/container for you.

### Types of Internal Stages

**a) User Stage (`@~`)** — every user automatically has one. Cannot be dropped, altered, or shared with other users.
```sql
PUT file://C:\data\employee.csv @~;
COPY INTO employee FROM @~;
```

**b) Table Stage (`@%table_name`)** — every table automatically has its own stage, tied 1:1 to that table. It is dropped automatically when the table is dropped.
```sql
PUT file://C:\data\employee.csv @%employee;
COPY INTO employee FROM @%employee;
```

**c) Named Internal Stage** — explicitly created, independent of any specific user/table; can be shared across users/roles via grants.
```sql
CREATE STAGE my_internal_stage;
PUT file://C:\data\employee.csv @my_internal_stage;
COPY INTO employee FROM @my_internal_stage;
```

---

## A3. External Stage

An **External Stage** points to storage *outside* Snowflake, in cloud storage you/your org own and manage:

- Amazon S3
- Microsoft Azure Blob Storage
- Google Cloud Storage (GCS)

```sql
CREATE STAGE my_external_stage
  URL = 's3://my-bucket/data/'
  STORAGE_INTEGRATION = my_s3_integration;

COPY INTO employee
FROM @my_external_stage;
```

> **Note:** Files are uploaded to external cloud storage directly (AWS CLI, S3 console, Azure Storage Explorer, etc.) — the `PUT` command does **not** work against external stages, since Snowflake doesn't own that storage.

---

## A4. Internal vs External — Comparison Table

| Feature | Internal Stage | External Stage |
|---|---|---|
| Storage location | Snowflake-managed storage | External cloud storage (your bucket) |
| Managed by | Snowflake | Cloud provider / customer |
| Examples | User, Table, Named Stage | S3, Azure Blob, GCS |
| File upload method | `PUT` command | Uploaded directly to cloud storage (not via `PUT`) |
| Access outside Snowflake | Not possible — Snowflake-only | Yes — other tools/apps can read the same files |
| Setup complexity | Simple, no extra setup | Needs storage integration + bucket permissions |
| Cost | Bundled into Snowflake storage billing | Billed separately by the cloud provider |
| Best for | Temporary, manual, dev/test loading | Enterprise-scale, automated, shared pipelines / data lakes |
| Storage Integration required | No | Commonly used (recommended for secure access) |
| Auto-created | User & Table stages: yes; Named stage: no | No — always explicitly created |
| Shareable across users/roles | Named internal stage only | Yes (via grants on the stage object) |
| Works with Snowpipe | Yes (less common) | Yes (most common pattern) |
| Data residency / compliance control | Limited — Snowflake controls it | Full control — useful for compliance needs |

---

## A5. Data Flow Diagrams

### Internal Stage
```text
Local File
    │  PUT
    ▼
Snowflake Internal Stage
    │  COPY INTO
    ▼
Snowflake Table
```
```sql
PUT file://C:\data\employee.csv @my_internal_stage;
COPY INTO employee FROM @my_internal_stage;
```

### External Stage
```text
Application / Source
        │
        ▼
Amazon S3 / Azure Blob / GCS
        │
        ▼
Snowflake External Stage
        │  COPY INTO
        ▼
Snowflake Table
```
```sql
COPY INTO employee FROM @my_external_stage;
```

---

## A6. When to Use Which

### Use an Internal Stage when:
- Files are small or temporary
- You want Snowflake to manage the storage
- You're manually loading files (e.g., via SnowSQL `PUT`)
- You're doing dev/test work
- You don't need/want a separate cloud storage account
- Nothing outside Snowflake needs to read the files

```sql
CREATE STAGE dev_stage;
PUT file://C:\data\test.csv @dev_stage;
COPY INTO test_table FROM @dev_stage;
```

### Use an External Stage when:
- Your org already stores files in S3 / Azure Blob / GCS
- You have large-scale, recurring pipelines
- Multiple applications/teams need access to the same files
- You want automated, event-driven ingestion (Snowpipe)
- Compliance requires control over exactly where data physically lives

```text
Application
     ↓
Cloud Storage (S3 / Azure / GCS)
     ↓
External Stage
     ↓
Snowpipe / COPY INTO
     ↓
Snowflake Table
```

### Rule of Thumb
> If Snowflake is the **only** consumer of the data → **Internal Stage**.
> If the data is shared across systems or part of a bigger data lake → **External Stage**.

---

## A7. Storage Integration

A **Storage Integration** is a Snowflake object that securely allows Snowflake to access external cloud storage, avoiding embedding cloud credentials (access keys/secrets) directly in SQL/stage definitions. It relies on cloud-provider IAM roles / service principals instead.

```sql
CREATE STORAGE INTEGRATION my_s3_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/my-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://my-bucket/data/');

CREATE STAGE my_external_stage
  URL = 's3://my-bucket/data/'
  STORAGE_INTEGRATION = my_s3_integration;
```

---

## A8. Simplified Recap

Think of a stage as a **"waiting area"** for files before they're loaded into a table (or after they're unloaded out of one).

- **Internal Stage** = storage *inside* Snowflake. Snowflake manages it completely — no cloud account of your own required.
- **External Stage** = a pointer to a bucket in *your own* cloud storage. Snowflake just "looks into" it; it doesn't own the storage.

**Analogy:**
- Internal stage = a locker inside Snowflake's building — only Snowflake has the key.
- External stage = your own storage unit outside — Snowflake has a key, but so can other tools/people.

---

## A9. Interview Q&A Bank (Stages)

**Q1. What is a stage in Snowflake?**
A location (internal or external) where data files are stored temporarily before loading into Snowflake tables, or after unloading from them.

**Q2. What are the types of internal stages?**
User stage (`@~`), Table stage (`@%table_name`), Named internal stage (`CREATE STAGE`).

**Q3. What's the main difference between internal and external stage?**
Internal stage storage is managed by Snowflake; external stage storage lives in the customer's own cloud storage (S3/Azure/GCS), and Snowflake only references it.

**Q4. Can you access files in an internal stage from outside Snowflake?**
No — only through Snowflake (SQL, SnowSQL, drivers, `PUT`/`GET`).

**Q5. What is a storage integration, and why is it needed for external stages?**
A Snowflake object holding a generated identity (IAM role/service principal) used to securely authenticate to cloud storage, so credentials aren't hardcoded in the stage definition.

**Q6. Which is cheaper — internal or external stage?**
Depends on scale. Internal costs are bundled into Snowflake storage pricing; external is billed directly by the cloud provider — often cheaper at large scale since you control storage tiers/lifecycle rules.

**Q7. Can the same external bucket be used by other tools besides Snowflake?**
Yes — a key advantage, since it's your own cloud bucket, other systems (Spark, other DBs, apps) can access the same files.

**Q8. How do you load data into Snowflake using a stage?**
```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV');
```

**Q9. What commands are used with internal stages but not external ones?**
`PUT` (upload local file to internal stage) and `GET` (download from internal stage). External stages don't use `PUT`/`GET` since files are managed directly via the cloud provider's own tools.

**Q10. Why would a company choose an external stage despite the extra setup?**
Compliance/data residency control, integration with an existing data lake, sharing files across multiple tools, and potentially lower cost at scale.

**Q11. Is data automatically encrypted in internal stages?**
Yes — Snowflake automatically encrypts data at rest in internal stages using Snowflake-managed keys.

**Q12. Can you list files in a stage?**
```sql
LIST @my_stage;
```

**Q13. What is the difference between a User Stage and a Table Stage?**
A User Stage belongs to a specific user (`@~`) and isn't tied to any table. A Table Stage belongs to a specific table (`@%table_name`) and is auto-dropped if the table is dropped.

**Q14. What is a Named Stage?**
A reusable stage explicitly created with `CREATE STAGE`. It can be internal or external, and — unlike user/table stages — its access can be granted to other roles/users.

**Q15. Can Snowpipe use a stage?**
Yes — Snowpipe loads files from a configured stage (typically external) into Snowflake tables automatically as new files arrive.
```text
Cloud Storage → External Stage → Snowpipe → Snowflake Table
```

---

## A10. Key Summary

```text
INTERNAL STAGE
---------------
→ Storage managed by Snowflake
→ User Stage, Table Stage, Named Internal Stage
→ Files uploaded via PUT
→ Best for temporary, dev/test, or manual loading

EXTERNAL STAGE
---------------
→ Storage outside Snowflake (S3 / Azure Blob / GCS)
→ Files uploaded directly to cloud storage (not via PUT)
→ Requires a Storage Integration for secure access (recommended)
→ Best for enterprise-scale, automated pipelines
→ Commonly paired with Snowpipe for continuous loading
```

### One-Line Answer
> An Internal Stage stores files in Snowflake-managed storage, whereas an External Stage references files stored in external cloud storage such as Amazon S3, Azure Blob Storage, or Google Cloud Storage.

---

# Part B — Iceberg Tables

## B1. Managed vs Self-Managed Iceberg Tables

Reference article (not reproduced here — summarized only, per copyright limits): the piece compares Snowflake's **Snowflake-managed Iceberg tables** (Snowflake owns the catalog and performs maintenance like compaction/cleanup) against **externally/self-managed Iceberg tables** (an external catalog, e.g. AWS Glue, owns metadata, and the customer or another engine handles maintenance) — the core distinction being *who controls the catalog and lifecycle operations*, which affects write access, multi-engine interoperability, and operational overhead.

- Source: [Snowflake Managed vs Self-Managed Iceberg Tables — What Actually Determines the Difference (Medium)](https://medium.com/snowflake/snowflake-managed-vs-self-managed-iceberg-tables-what-actually-determines-the-difference-c5615ba0d280)

> This topic wasn't detailed in your source notes beyond the link — happy to write a full breakdown (catalog options, when to pick managed vs. externally-managed, trade-offs) as a separate section if useful.

---

# Part C — Real-Time Scenarios (Set 1)

*Source: scenarios based on Vishal Kaushal-style walkthroughs. Each follows: Problem → Idea → Code → Explanation.*

## C1. Dynamic S3 Date-Folder Loads

### Problem
Files land in S3 under a date-partitioned folder structure that changes daily:
```
snow_aws_project / 2025 / 06 / 20 / customer.csv
```
The path can't be hardcoded — it must be built dynamically from the current date.

### Idea
1. Extract year/month/day from the current date.
2. Concatenate into a folder path (`2025/06/20/`).
3. Use it inside a dynamically-built `COPY INTO`.
4. Wrap it all in a stored procedure (later schedulable via a Task).

### Code
```sql
-- Target table structure only (no rows)
CREATE OR REPLACE TABLE CUSTOMER AS
SELECT * FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER WHERE 1=2;

CREATE OR REPLACE PROCEDURE DYNAMIC_LOAD()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
    yr STRING;
    mn STRING;
    dy STRING;
    full_path STRING;
    copy_sql STRING;
BEGIN
    yr := TO_CHAR(CURRENT_DATE, 'YYYY');
    mn := TO_CHAR(CURRENT_DATE, 'MM');
    dy := TO_CHAR(CURRENT_DATE, 'DD');

    full_path := yr || '/' || mn || '/' || dy || '/';

    copy_sql := 'COPY INTO CUSTOMER FROM @DYNAMIC_STAGE/' || full_path;

    EXECUTE IMMEDIATE :copy_sql;

    RETURN copy_sql;
END;
$$;

CALL DYNAMIC_LOAD();
```

### Explanation

| Code Part | What It Does |
|---|---|
| `WHERE 1=2` | Copies only table *structure*, no rows |
| `DECLARE ... yr, mn, dy` | Temporary variables for year/month/day |
| `TO_CHAR(CURRENT_DATE, 'YYYY')` | Extracts just the year, e.g. `2025` |
| `full_path := yr \|\| '/' \|\| mn \|\| '/' \|\| dy \|\| '/'` | Joins parts into a folder path like `2025/06/20/` |
| `copy_sql := 'COPY INTO ...' \|\| full_path` | Builds the SQL text since the path isn't known until runtime |
| `EXECUTE IMMEDIATE :copy_sql` | Runs the built text as real SQL (**Dynamic SQL**) |
| `RETURN copy_sql` | Returns the executed SQL text for audit/debugging |

**Prerequisite objects** (assumed pre-existing): a File Format, a Storage Integration, and a Stage pointing at the *top-level* S3 folder only (no year/month/day) — kept generic so it works for every future date automatically.

### ⚠️ Gotcha
Calling the procedure twice won't reload data — `COPY INTO` skips already-processed files. To reload: `TRUNCATE TABLE CUSTOMER` first, or add `FORCE = TRUE`.

### Bonus Variation
If files sit in one shared folder named like `customer_20250621.csv` instead of date-folders, build a **file name pattern** instead and pass it to `COPY INTO ... PATTERN = '...'`, using the same date-extraction logic.

---

## C2. Valid Data Load + Auto-Send Failures to Client

### Problem
Client CSVs contain both good and bad rows. Requirements: (1) load good rows into the main table, (2) capture failed rows separately, (3) export failed rows back to S3 for the client to fix.

### Idea
`ON_ERROR = CONTINUE` skips bad rows during `COPY INTO` while still loading good ones. Snowflake's `VALIDATE()` table function then reports exactly which rows failed and why from the last `COPY INTO` run. A reverse `COPY INTO` (table → S3) sends the bad rows back out.

### Code

**Setup:**
```sql
CREATE OR REPLACE TABLE CUSTOMER (...);

CREATE OR REPLACE FILE FORMAT CSVTYPE
TYPE='CSV' SKIP_HEADER=1 FIELD_DELIMITER=','
RECORD_DELIMITER='\n' FIELD_OPTIONALLY_ENCLOSED_BY='"'
DATE_FORMAT='DD-MM-YYYY' COMPRESSION=NONE;

CREATE OR REPLACE STORAGE INTEGRATION AWS_INT ...;

CREATE OR REPLACE STAGE AWS_STAGE ...;         -- source folder
CREATE OR REPLACE STAGE AWS_TARGET_STAGE ...;  -- rejected-files folder
```

**Load, skipping bad rows:**
```sql
COPY INTO CUSTOMER
FROM @AWS_STAGE
ON_ERROR = CONTINUE;
```

**Find exactly which rows failed:**
```sql
SELECT * FROM TABLE(VALIDATE(CUSTOMER, JOB_ID => '_last'));
```
`VALIDATE()` inspects the **most recent** `COPY INTO` run on the table (`JOB_ID => '_last'`) and lists every rejected row with error reason, file name, and row number.

**Save failed rows into their own table:**
```sql
CREATE OR REPLACE TABLE FAULTY_RECORDS AS
SELECT FILE, CATEGORY, ERROR, REJECTED_RECORD
FROM TABLE(VALIDATE(CUSTOMER, JOB_ID => '_last'));
```

**Send failed rows back to S3:**
```sql
COPY INTO @AWS_TARGET_STAGE/faultyrecords.csv
FROM FAULTY_RECORDS
SINGLE = TRUE;
```
`SINGLE = TRUE` forces one output file instead of Snowflake's default multi-file split.

### Reusable Stored Procedure
```sql
CREATE OR REPLACE PROCEDURE FAULTY_RECORDS_EXPORT(
    TABLE_NAME STRING,
    SOURCE_STAGE STRING,
    TARGET_STAGE STRING
)
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    LET COPY_SQL := '
        COPY INTO ' || TABLE_NAME || '
        FROM @' || SOURCE_STAGE || '
        ON_ERROR = CONTINUE;
    ';
    EXECUTE IMMEDIATE :COPY_SQL;

    LET VALIDATE_SQL := '
        CREATE OR REPLACE TABLE FAULTY_RECORDS AS
        SELECT FILE, CATEGORY, ERROR, REJECTED_RECORD
        FROM TABLE(
            VALIDATE(' || TABLE_NAME || ', JOB_ID => ''_last'')
        );
    ';
    EXECUTE IMMEDIATE :VALIDATE_SQL;

    LET EXPORT_SQL := '
        COPY INTO @' || TARGET_STAGE || '/faultyrecords.csv
        FROM FAULTY_RECORDS
        SINGLE = TRUE
        OVERWRITE = TRUE;
    ';
    EXECUTE IMMEDIATE :EXPORT_SQL;

    RETURN 'Load and export completed successfully.';
END;
$$;

CALL FAULTY_RECORDS_EXPORT('CUSTOMER', 'AWS_STAGE', 'AWS_TARGET_STAGE');
```

**Why parameters?** So one procedure serves the Product table, Order table, Inventory table, etc. — not one procedure per table.

**Why 3 separate `EXECUTE IMMEDIATE` blocks?** Each step (load → validate/save → export) needs table/stage names built dynamically as text first, since they're parameters, not fixed identifiers.

### Automating Further
Wrap the procedure call inside a **Task** scheduled at fixed times (e.g. 12 PM, 5 PM, 8 PM) for hands-off operation.

---

## C3. Schema Evolution

### Problem
A pipeline loads a client's CSV into a table. When the client adds/reorders columns, the load normally breaks because the file no longer matches the table.

### Idea
- **Schema Detection**: Snowflake can infer column names/types from a file (`INFER_SCHEMA`).
- **Schema Evolution**: Once a table exists, Snowflake can automatically add new columns when a differently-shaped file is loaded — no manual `ALTER TABLE`.

**Limitation:** only works with `COPY INTO` / Snowpipe — not plain `INSERT`.

### Code

**File format (needs `PARSE_HEADER`, not `SKIP_HEADER`):**
```sql
CREATE OR REPLACE FILE FORMAT CSVTYPE
TYPE='CSV'
PARSE_HEADER=TRUE
RECORD_DELIMITER='\n'
FIELD_DELIMITER=','
FIELD_OPTIONALLY_ENCLOSED_BY='"'
ENCODING='UTF8';
```

**Storage integration + stage (standard):**
```sql
CREATE OR REPLACE STORAGE INTEGRATION S3_INT ...;
CREATE OR REPLACE STAGE SCHEMA_DETECT ...;
LIST @SCHEMA_DETECT;
```

**Detect the schema:**
```sql
SELECT * FROM TABLE(
    INFER_SCHEMA(
        LOCATION => '@SCHEMA_DETECT/customer_file.csv',
        FILE_FORMAT => 'CSVTYPE',
        IGNORE_CASE => TRUE
    )
);
```
`INFER_SCHEMA()` reads the file and returns column names + guessed data types; `IGNORE_CASE => TRUE` ignores casing differences.

**Auto-create a table from the inferred schema:**
```sql
CREATE OR REPLACE TABLE CUSTOMER_DATA
USING TEMPLATE (
    SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*)) FROM TABLE(
        INFER_SCHEMA(
            LOCATION => '@SCHEMA_DETECT/customer_file.csv',
            FILE_FORMAT => 'CSVTYPE',
            IGNORE_CASE => TRUE
        )
    )
);
```
`USING TEMPLATE` builds table columns from the JSON-like inferred-schema structure; `ARRAY_AGG(OBJECT_CONSTRUCT(*))` packages all inferred column definitions into one array.

**Load the first (matching) file:**
```sql
COPY INTO CUSTOMER_DATA
FROM @SCHEMA_DETECT/customer_us_file.csv
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```
`MATCH_BY_COLUMN_NAME` matches file→table columns by name (not position), so reordered columns still work. `CASE_INSENSITIVE` ignores case.

**Enable Schema Evolution before loading a file with an EXTRA column:**
```sql
ALTER TABLE CUSTOMER_DATA SET ENABLE_SCHEMA_EVOLUTION = TRUE;
ALTER FILE FORMAT CSVTYPE SET ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE;
```
- `ENABLE_SCHEMA_EVOLUTION = TRUE` → table is allowed to auto-gain columns.
- `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` → file format won't error on a different column count.

**Load the new (wider) file:**
```sql
COPY INTO CUSTOMER_DATA
FROM @SCHEMA_DETECT/customer_us_file.csv
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```
Snowflake automatically adds the new column (e.g. `AUDIT_DATE`) to the table and loads the data; older rows show `NULL` for it.

### Key Takeaways

| Setting | Purpose |
|---|---|
| `INFER_SCHEMA` | Auto-detects column names/types from a file |
| `USING TEMPLATE` | Auto-creates a table from detected schema |
| `MATCH_BY_COLUMN_NAME` | Matches file↔table columns by name (handles reordering) |
| `ENABLE_SCHEMA_EVOLUTION` | Lets the table auto-add new columns |
| `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` | Stops rejects due to column-count mismatch |

---

## C4. Dynamic Multi-Folder → Multi-Table Ingestion

### Problem
An S3 bucket has multiple folders (`customer/`, `nation/`, `region/`, `orders/`...), each needing its own matching table. New folders (e.g. `supplier/`) must automatically get a new table + data load, with zero code changes.

### Idea
1. Enable a **Directory Table** on the stage (queryable list of file paths via `RELATIVE_PATH`).
2. Loop through file paths with a **CURSOR**.
3. Extract folder name (→ table name) and file name from each path.
4. Check an audit table to skip already-loaded files.
5. Auto-create the table via `INFER_SCHEMA` if it doesn't exist.
6. Load via `COPY INTO ... MATCH_BY_COLUMN_NAME`.
7. Log the file as processed.

### Code

**Setup:**
```sql
CREATE OR REPLACE SCHEMA BRONZE;
CREATE OR REPLACE FILE FORMAT CSVONE CLONE RAWLAYER.CSVTYPE;
CREATE OR REPLACE STORAGE INTEGRATION DYNAMIC_INT ...;

CREATE OR ALTER STAGE DYNAMICLOAD
FILE_FORMAT = CSVONE
STORAGE_INTEGRATION = DYNAMIC_INT
URL = 's3://realtimeproject-snowflake/'
DIRECTORY = (ENABLE = true, AUTO_REFRESH = TRUE);
```
`DIRECTORY = (ENABLE = true)` turns on a hidden table tracking every file's exact path. `AUTO_REFRESH = TRUE` keeps it current automatically (via S3 event notifications behind the scenes).

**Query the directory table:**
```sql
SELECT RELATIVE_PATH FROM DIRECTORY(@DYNAMICLOAD);
```
Returns rows like `customer/customer.csv`, `nation/nation.csv`, `region/region.csv` — folder name = table name, filename = the rest.

**Audit table:**
```sql
CREATE OR REPLACE TABLE FILE_LOAD_LOG (
    FILE_NAME STRING,
    LOAD_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Full stored procedure:**
```sql
CREATE OR REPLACE PROCEDURE DYNAMIC_TABLE_LOAD()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
    file_path STRING;
    folder_name STRING;
    tablename STRING;
    file_name STRING;
    full_stage_path STRING;
    record_count INTEGER;
    file_cursor CURSOR FOR SELECT RELATIVE_PATH FROM DIRECTORY(@DYNAMICLOAD);

BEGIN
    FOR file_rec IN file_cursor DO
        file_path := file_rec.RELATIVE_PATH;              -- e.g. customer/customer.csv
        folder_name := SPLIT_PART(file_path, '/', 1);      -- customer
        file_name := SPLIT_PART(file_path, '/', 2);        -- customer.csv

        SELECT COUNT(1) INTO :record_count FROM FILE_LOAD_LOG WHERE FILE_NAME = :file_path;
        IF (record_count > 0) THEN
            CONTINUE;
        END IF;

        tablename := UPPER(folder_name);
        full_stage_path := '@DYNAMICLOAD/' || file_path;

        SELECT COUNT(1) INTO :record_count
        FROM INFORMATION_SCHEMA.TABLES
        WHERE TABLE_SCHEMA = CURRENT_SCHEMA()
          AND TABLE_NAME = :tablename;

        IF (record_count = 0) THEN
            LET create_sql := '
                CREATE OR REPLACE TABLE IDENTIFIER(?)
                USING TEMPLATE (
                    SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
                    FROM TABLE(
                        INFER_SCHEMA(
                            LOCATION => ?,
                            FILE_FORMAT => ''CSVONE'',
                            IGNORE_CASE => TRUE
                        )
                    )
                )';
            EXECUTE IMMEDIATE create_sql USING (tablename, full_stage_path);
        END IF;

        LET copy_sql := '
            COPY INTO IDENTIFIER(?)
            FROM ?
            FILE_FORMAT = (FORMAT_NAME = ''CSVONE'')
            MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
            ON_ERROR = ''SKIP_FILE''';
        EXECUTE IMMEDIATE copy_sql USING (tablename, full_stage_path);

        INSERT INTO FILE_LOAD_LOG (FILE_NAME) VALUES (:file_path);
    END FOR;

    RETURN 'Dynamic ingestion from S3 to Snowflake complete.';
END;
$$;

CALL DYNAMIC_TABLE_LOAD();
```

### Explanation

| Code Part | Meaning |
|---|---|
| `CURSOR FOR SELECT RELATIVE_PATH FROM DIRECTORY(@DYNAMICLOAD)` | Prepares a list of every file path to iterate |
| `FOR file_rec IN file_cursor DO ... END FOR` | Loops over every file path |
| `SPLIT_PART(file_path, '/', 1)` | Grabs the folder name |
| `SPLIT_PART(file_path, '/', 2)` | Grabs the file name |
| `IF (record_count > 0) THEN CONTINUE; END IF;` | Skips already-logged files |
| `IDENTIFIER(?)` with `USING (...)` | Safely plugs a variable table name into SQL |
| `ON_ERROR = 'SKIP_FILE'` | Skips an entire failed file without stopping the procedure |
| `INSERT INTO FILE_LOAD_LOG` | Marks the file as processed |

### Why This Is Powerful
- New folder (e.g. `supplier/`) → next run auto-creates a `SUPPLIER` table and loads data, zero code changes.
- `FILE_LOAD_LOG` prevents duplicate loads on repeat runs.
- `AUTO_REFRESH = TRUE` + S3 event notifications keep the directory table current.

---

## C5. Dynamic Table Creation from Single-Cell CSV Headers

### Problem
Instead of normal CSV columns, all header names are crammed into **one cell**, pipe-delimited:
```
customer_id|customer_name|age|phone_number|address|email
1001,John,25,9999999999,Delhi,john@test.com
```
`INFER_SCHEMA` can't parse this — the header must be parsed manually.

### Idea
1. Get the file name from the stage's directory table → derive the table name.
2. Read the header line (`$1`) into a temp table.
3. Split by `|` into an array of column names.
4. Get array size (column count).
5. Loop to build a column-definition string.
6. Dynamically build/execute `CREATE TABLE`.
7. Load the actual data with `COPY INTO`, skipping the header row.

### Code
```sql
CREATE OR REPLACE PROCEDURE CREATE_DYNAMIC_TABLE_FROM_HEADER()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
    file_name STRING;
    table_name STRING;
    header_line STRING;
    column_list ARRAY;
    column_count INT;
    column_definitions STRING DEFAULT '';
    i INT DEFAULT 0;
    col_def STRING;
    create_table_sql STRING;
    copy_into_sql STRING;

BEGIN
    SELECT RELATIVE_PATH INTO file_name FROM DIRECTORY(@HEADER);

    table_name := UPPER(STRTOK(file_name, '.', 1));

    EXECUTE IMMEDIATE '
        CREATE OR REPLACE TEMP TABLE TMP_HEADER_LINE AS
        SELECT $1 AS HEADER_LINE
        FROM @HEADER/' || file_name || '
        (FILE_FORMAT => (CSVTYPE))
        LIMIT 1
    ';

    SELECT HEADER_LINE INTO header_line FROM TMP_HEADER_LINE;

    column_list := SPLIT(header_line, '|');
    column_count := ARRAY_SIZE(column_list);

    WHILE (i < column_count) DO
        BEGIN
            LET col_name := TRIM(column_list[i]);
            col_def := '"' || col_name || '" STRING';

            IF (i > 0) THEN
                column_definitions := column_definitions || ', ';
            END IF;

            column_definitions := column_definitions || col_def;
            i := i + 1;
        END;
    END WHILE;

    create_table_sql := 'CREATE OR REPLACE TABLE "' || table_name || '" (' || column_definitions || ')';
    EXECUTE IMMEDIATE create_table_sql;

    copy_into_sql := 'COPY INTO "' || table_name || '" FROM @HEADER/' || file_name ||
                      ' FILE_FORMAT = (TYPE = CSV, FIELD_DELIMITER = '','', SKIP_HEADER = 1)';
    EXECUTE IMMEDIATE copy_into_sql;

    RETURN 'Table "' || table_name || '" created successfully and data also get loaded successfully from file "' || file_name || '".';
END;
$$;

CALL CREATE_DYNAMIC_TABLE_FROM_HEADER();
```

### Explanation

| Code Part | What It Does |
|---|---|
| `SELECT RELATIVE_PATH INTO file_name FROM DIRECTORY(@HEADER)` | Grabs the file's path/name |
| `STRTOK(file_name, '.', 1)` | `customerheader.csv` → `customerheader` (strips extension) |
| `SELECT $1 AS HEADER_LINE FROM @HEADER/... LIMIT 1` | `$1` = entire first cell as raw text (the crammed header) |
| `SPLIT(header_line, '|')` | Breaks it into a proper array of column names |
| `ARRAY_SIZE(column_list)` | Counts columns |
| `WHILE (i < column_count) DO ... END WHILE` | Loops once per column (0-indexed) |
| `TRIM(column_list[i])` | Removes stray spaces |
| `col_def := '"' \|\| col_name \|\| '" STRING'` | Builds e.g. `"customer_id" STRING` |
| `IF (i > 0) THEN column_definitions := column_definitions \|\| ', '; END IF;` | Adds commas correctly between columns |
| `CREATE OR REPLACE TABLE "..." (...)` | Builds the final `CREATE TABLE` |
| `COPY INTO ... SKIP_HEADER = 1` | Skips the header row now that columns are already known |

### Key Takeaway
This is fundamentally **manual string parsing + dynamic SQL** for when `INFER_SCHEMA` can't understand a non-standard file layout.

---

## C6. Tracking DDL Changes (Schema Audit Log)

### Problem
Snowflake tracks **DML** changes (INSERT/UPDATE/DELETE) easily via Streams, but has no built-in way to track **DDL/structural** changes — added/dropped columns, changed data types. Need a full audit trail of *what changed, when*.

### Idea
A "diff" pattern — compare a "before" snapshot against the "after" (live) state:
1. Keep a snapshot table of current tables/columns/types.
2. Compare it against live `INFORMATION_SCHEMA.COLUMNS`.
3. In "after" but not "before" → **ADDED**.
4. In "before" but not "after" → **DROPPED**.
5. In both but different type → **MODIFIED**.
6. Log differences, then refresh the snapshot.

### Code

**Initial snapshot:**
```sql
CREATE OR REPLACE TABLE CURRENT_SNAPSHOT AS
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_CATALOG = 'YOUTUBELEARNING' AND TABLE_SCHEMA = 'BRONZE';
```

**Audit/log table:**
```sql
CREATE OR REPLACE TABLE DDL_CHANGE_LOG (
    TABLE_NAME STRING,
    COLUMN_NAME STRING,
    OPERATION_MADE STRING,     -- 'ADDED', 'DROPPED', 'MODIFIED'
    OLD_DATA_TYPE STRING,
    NEW_DATA_TYPE STRING,
    CHANGE_TIMESTAMP TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    CHANGE_BY STRING
);
```

**Stored procedure:**
```sql
CREATE OR REPLACE PROCEDURE SP_TRACK_DDL_CHANGES()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    -- ADDED
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        c.TABLE_NAME, c.COLUMN_NAME, 'ADDED', NULL, c.DATA_TYPE,
        CURRENT_TIMESTAMP(), CURRENT_USER()
    FROM INFORMATION_SCHEMA.COLUMNS c
    LEFT JOIN CURRENT_SNAPSHOT s
        ON c.TABLE_NAME = s.TABLE_NAME AND c.COLUMN_NAME = s.COLUMN_NAME
    WHERE c.TABLE_CATALOG = 'YOUTUBELEARNING'
      AND c.TABLE_SCHEMA = 'BRONZE'
      AND s.COLUMN_NAME IS NULL;

    -- DROPPED
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        s.TABLE_NAME, s.COLUMN_NAME, 'DROPPED', s.DATA_TYPE, NULL,
        CURRENT_TIMESTAMP(), CURRENT_USER()
    FROM CURRENT_SNAPSHOT s
    LEFT JOIN INFORMATION_SCHEMA.COLUMNS c
        ON s.TABLE_NAME = c.TABLE_NAME AND s.COLUMN_NAME = c.COLUMN_NAME
        AND c.TABLE_CATALOG = 'YOUTUBELEARNING' AND c.TABLE_SCHEMA = 'BRONZE'
    WHERE c.COLUMN_NAME IS NULL;

    -- MODIFIED
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        c.TABLE_NAME, c.COLUMN_NAME, 'MODIFIED', s.DATA_TYPE, c.DATA_TYPE,
        CURRENT_TIMESTAMP(), CURRENT_USER()
    FROM INFORMATION_SCHEMA.COLUMNS c
    JOIN CURRENT_SNAPSHOT s
        ON c.TABLE_NAME = s.TABLE_NAME AND c.COLUMN_NAME = s.COLUMN_NAME
    WHERE c.TABLE_CATALOG = 'YOUTUBELEARNING'
      AND c.TABLE_SCHEMA = 'BRONZE'
      AND c.DATA_TYPE <> s.DATA_TYPE;

    -- Refresh the snapshot
    DELETE FROM CURRENT_SNAPSHOT;
    INSERT INTO CURRENT_SNAPSHOT
    SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_CATALOG = 'YOUTUBELEARNING' AND TABLE_SCHEMA = 'BRONZE';

    RETURN 'DDL change detection completed successfully.';
END;
$$;

CALL SP_TRACK_DDL_CHANGES();
```

### Join Logic Explained

| Detection | Join Used | Why |
|---|---|---|
| **ADDED** | Live `LEFT JOIN` Snapshot | Start from live data; if no snapshot match (`s.COLUMN_NAME IS NULL`), the column is new |
| **DROPPED** | Snapshot `LEFT JOIN` Live | Start from old snapshot; if no live match (`c.COLUMN_NAME IS NULL`), the column was removed |
| **MODIFIED** | `INNER JOIN` on both | Only columns present in both, but with differing `DATA_TYPE` |

**Why refresh the snapshot?** Without refreshing, re-running the procedure would re-detect and re-log the same changes. Refreshing keeps future comparisons accurate to only *new* changes.

### Key Takeaway
This is a general **"diff" pattern** (also used in version control, data reconciliation, etc.): save a "before" state, compare to "now," log the differences.

---

## C7. Summary Table — Set 1

| # | Scenario | Core Feature | Key Skill Tested |
|---|---|---|---|
| C1 | Dynamic S3 date-folder loading | Stored Procedures, Dynamic SQL, `EXECUTE IMMEDIATE` | Building dynamic paths from the current date |
| C2 | Valid vs. faulty record routing | `ON_ERROR = CONTINUE`, `VALIDATE()`, reverse `COPY INTO` | Graceful error handling for bad data |
| C3 | Schema Evolution | `INFER_SCHEMA`, `USING TEMPLATE`, `ENABLE_SCHEMA_EVOLUTION` | Handling changing file structures automatically |
| C4 | Multi-folder → multi-table pipeline | Directory Tables, Cursors, `IDENTIFIER()`, audit logging | Fully dynamic, scalable ingestion |
| C5 | Single-cell header parsing | `SPLIT()`, `ARRAY_SIZE()`, `WHILE` loops, dynamic `CREATE TABLE` | Manual parsing when built-in tools don't apply |
| C6 | DDL Change Tracking | `INFORMATION_SCHEMA.COLUMNS`, snapshot comparison, `LEFT JOIN` | Building a diff-based schema audit trail |

**Common thread:** Stored Procedures + Dynamic SQL (`EXECUTE IMMEDIATE`) drive nearly every automation scenario — they let you build SQL as text at runtime whenever table names, paths, or column lists aren't known ahead of time.

---

# Part D — Real-Time Scenarios (Set 2)

## D1. Load + Validate in a Single Stored Procedure

### Problem
A client sends a CSV daily. The old way: load raw → discover bad data later → manually clean/reload — wastes hours. **Better:** validate before committing — load into a temp staging table, run checks, only push to the real table if it passes, and always log the outcome.

### Design
```
Stage File (CSV)
     │
     ▼
Temporary Table  ──► Checks: nulls? duplicates? zero rows?
     │
     ├── ✅ PASS ──► Insert into Final Table
     └── ❌ FAIL ──► Don't insert, just log the reason
     │
     ▼
Always write outcome to AUDIT_LOG
```
All wrapped in **one Stored Procedure**.

### Code
```sql
CREATE OR REPLACE PROCEDURE LOAD_AND_VALIDATE()
RETURNS VARCHAR()
LANGUAGE SQL
AS
$$
DECLARE
    temp_table_name STRING;
    total_rows_loaded INT DEFAULT 0;
    duplicate_rows INT DEFAULT 0;
    null_rows INT DEFAULT 0;
    status VARCHAR(50);
    error_message VARCHAR();
    result_message STRING;

BEGIN
    temp_table_name := 'TEMP_CUST_DATA';
    status := 'Failed';
    error_message := NULL;

    CREATE OR REPLACE TEMPORARY TABLE IDENTIFIER(:temp_table_name) (
       C_CUSTKEY    INT,
       C_NAME       STRING,
       C_ADDRESS    STRING,
       C_NATIONKEY  INT,
       C_PHONE      STRING,
       C_ACCTBAL    INT,
       C_MKTSEGMENT STRING,
       C_COMMENT    STRING
    );

    BEGIN
        COPY INTO IDENTIFIER(:temp_table_name)
        FROM @SOURCE_DATA 
        FILE_FORMAT = (FORMAT_NAME='CSVONE')
        ON_ERROR = 'ABORT_STATEMENT';

        total_rows_loaded := (SELECT COUNT(*) FROM IDENTIFIER(:temp_table_name));

        null_rows := (
            SELECT COUNT(*) FROM IDENTIFIER(:temp_table_name)
            WHERE C_PHONE IS NULL OR C_NATIONKEY IS NULL
        );

        duplicate_rows := (
            SELECT COUNT(*)
            FROM (
                SELECT C_CUSTKEY,
                       ROW_NUMBER() OVER (PARTITION BY C_CUSTKEY ORDER BY C_CUSTKEY) AS rn
                FROM IDENTIFIER(:temp_table_name)
            ) a
            WHERE rn > 1
        );

        IF (null_rows = 0 AND duplicate_rows = 0 AND total_rows_loaded > 0) THEN
            INSERT INTO CUSTOMER_DATA (
                C_CUSTKEY, C_NAME, C_ADDRESS, C_NATIONKEY, C_PHONE, C_ACCTBAL, C_MKTSEGMENT, C_COMMENT
            )
            SELECT C_CUSTKEY, C_NAME, C_ADDRESS, C_NATIONKEY, C_PHONE, C_ACCTBAL, C_MKTSEGMENT, C_COMMENT  
            FROM IDENTIFIER(:temp_table_name);

            status := 'Success';
            result_message := 'Successfully loaded ' || total_rows_loaded || ' rows with no validation errors.';
        ELSE
            status := 'Failed - Validation Errors';
            error_message := 'Validation failed. ' || null_rows || ' rows with nulls, ' || duplicate_rows || ' duplicate rows found.';
            result_message := error_message;
        END IF;

    EXCEPTION
        WHEN OTHER THEN
            status := 'Failed - Execution Error';
            error_message := SQLERRM;
            result_message := error_message;
    END;

    INSERT INTO AUDIT_LOG (
        status, load_timestamp, rows_loaded, duplicate_rows, null_rows, error_message
    )
    VALUES (
        :status, CURRENT_TIMESTAMP(), :total_rows_loaded, :duplicate_rows, :null_rows, :error_message
    );

    RETURN 'Procedure execution completed with status: ' || status || '. Message: ' || result_message;

END;
$$;

CALL LOAD_AND_VALIDATE();
```

### Explanation

**1. Procedure declaration** — a Snowflake Scripting stored procedure, no parameters, returns a `VARCHAR` summary.

**2. Variable declarations** — local "memory boxes" for the run; row counters default to `0` so they're never `NULL` even if skipped.

**3. Initial defaults** — `status := 'Failed'` upfront is defensive coding: if something unexpected happens and nothing else runs, you still log "Failed" instead of blank.

**4. Temp table via `IDENTIFIER(:temp_table_name)`** — lets the table name come from a variable (reusable pattern); `TEMPORARY` means it exists only for the session and auto-disappears, ideal as a scratch pad.

**5. Nested `BEGIN...EXCEPTION`** — like `try/catch`. Anything risky (load, validation) sits inside; `EXCEPTION WHEN OTHER` catches any error (e.g. malformed CSV breaking `COPY INTO`) instead of crashing the whole procedure. `SQLERRM` auto-captures Snowflake's actual error text. **This is the key design choice** — the procedure never dies silently.

**6. Load step:**
```sql
COPY INTO IDENTIFIER(:temp_table_name)
FROM @SOURCE_DATA 
FILE_FORMAT = (FORMAT_NAME='CSVONE')
ON_ERROR = 'ABORT_STATEMENT';
```
`ON_ERROR = 'ABORT_STATEMENT'` — zero tolerance for structurally broken files; one bad row halts the load, caught by the exception block.

**7. Validation checks:**
- **Row count** — sanity check that any data arrived at all.
- **Null check** — counts rows missing mandatory fields (`C_PHONE`, `C_NATIONKEY`).
- **Duplicate check** — the classic `ROW_NUMBER() OVER (PARTITION BY ...)` pattern: partition by key, number within each group, any `rn > 1` is a repeat.

**8. Insert-or-reject gate** — data moves to `CUSTOMER_DATA` only if *all three* conditions pass (no nulls, no dupes, rows > 0); otherwise nothing is inserted and a descriptive failure message is built.

**9. Unconditional audit log insert** — runs regardless of outcome, because it sits *outside* the inner exception block — guarantees a full run history.

**10. Final summary return** — a single readable string for whoever/whatever called the procedure.

### What Was Demonstrated
1. Ran with a broken file format → *execution error* → logged as `Failed - Execution Error`.
2. Fixed the format, ran again → COPY succeeded, but 2 null rows + 1 duplicate caught → `Failed - Validation Errors`, **zero rows inserted**.
3. Fixed the source data, re-ran → all checks passed → **126,000+ rows inserted**, logged `Success`.

### Interview Takeaways

| Concept | Why It Matters |
|---|---|
| `IDENTIFIER(:var)` | Dynamically reference table/object names via variables — reusable procedures |
| `TEMPORARY` table | Safe, self-cleaning scratch space for staging/validation |
| Nested `BEGIN...EXCEPTION` | Mimics try/catch; guarantees a status is always produced |
| `SQLERRM` | Captures the actual system error inside an exception handler |
| `ROW_NUMBER() OVER (PARTITION BY ...)` | Standard SQL duplicate-detection pattern |
| Audit logging outside the try block | Guarantees a log entry regardless of outcome |
| "Validate before commit" pattern | Catch issues at ingestion, not after |

---

## D2. Real-Time Pipeline — Dynamic Tables + Alerts

### Problem
An S3 bucket continuously receives customer/item files. Requirements: (1) ingest into a raw/staging layer, (2) build a clean layer (latest customer record; highest-price item), (3) combine into a final reporting table with a derived metric, (4) auto-email on calculation failure (e.g. divide-by-zero) instead of relying on manual monitoring.

Built using **Dynamic Tables** — Snowflake automatically keeps downstream tables refreshed as source data changes.

### Design
```
S3 Bucket (customer.csv, item.csv)
        │  (Snowpipe ingestion — assumed)
        ▼
STG_CUSTOMER, ITEM   (raw staging tables)
        │
        ▼
CUSTOMER_DT   (dedupe → latest record per customer)
ITEM_DT       (dedupe → highest-price item per customer)
        │
        ▼
CUST_ITEM_DT  (join + calculate price per item)
        │
        ▼
If refresh fails → ALERT checks refresh history every 1 min
        │
        ▼
   Sends email via Notification Integration
```

### Code

**Staging tables + sample data:**
```sql
CREATE OR REPLACE SCHEMA PIPELINE;

CREATE OR REPLACE TABLE STG_Customer (
    CUST_ID STRING,
    CUST_NAME STRING,
    OUTSTANDING_AMT NUMBER,
    CRID STRING,
    LOCATION STRING,
    CUST_CREATED DATE
);

INSERT INTO STG_Customer (CUST_ID, CUST_NAME, OUTSTANDING_AMT, CRID, LOCATION, CUST_CREATED) VALUES
('C-101', 'Raman', 500, 'ABVC', 'LA', '2025-08-11'),
('C-101', 'Raman', 500, 'ABVC', 'LA', '2025-08-12'), -- cleansed layer should keep latest
('C-102', 'Rahul', 200, 'XYZ', 'AF', '2025-08-14'),
('C-103', 'Anshi', 5000, 'MNCD', 'GA', '2025-08-15');

CREATE OR REPLACE TABLE ITEM (
    CUST_ID STRING,
    ITEM_ID STRING,
    ITEM_CATEGORY STRING,
    ITEM_STATUS STRING,
    COUNTS NUMBER,
    PRICE NUMBER
);

INSERT INTO ITEM (CUST_ID, ITEM_ID, ITEM_CATEGORY, ITEM_STATUS, COUNTS, PRICE) VALUES
('C-101', 'a-101', 'Printer', 'Active', 1, 100),
('C-101', 'a-101', 'Printer', 'Active', 4, 200), -- kept
('C-102', 'a-103', 'Ink', 'Active', 2, 300),
('C-103', 'a-103', 'Ribbon', 'Active', 3, 100),
('C-103', 'a-103', 'Ribbon', 'Active', 2, 200); -- highest price, kept
```

**Cleansed-layer dynamic tables:**
```sql
CREATE OR REPLACE DYNAMIC TABLE CUSTOMER_DT
  TARGET_LAG = DOWNSTREAM
  WAREHOUSE = COMPUTE_WH
  INITIALIZE = ON_CREATE
AS
  SELECT * FROM STG_CUSTOMER 
  QUALIFY ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY CUST_CREATED DESC) = 1;

CREATE OR REPLACE DYNAMIC TABLE ITEM_DT
  TARGET_LAG = DOWNSTREAM
  WAREHOUSE = COMPUTE_WH
AS
  SELECT * FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY PRICE DESC) AS rn
    FROM ITEM 
  ) t
  WHERE rn = 1;
```

**Final combined dynamic table:**
```sql
CREATE OR REPLACE DYNAMIC TABLE CUST_ITEM_DT
  TARGET_LAG = '1 MINUTES'
  WAREHOUSE = COMPUTE_WH
AS
  SELECT c.cust_id, c.cust_name, c.crid, c.location, c.cust_created,
         a.item_id, a.item_category, a.item_status, a.price, a.counts,
         ROUND(a.price / a.counts, 2) AS Price_Per_item
  FROM CUSTOMER_DT c, ITEM_DT a
  WHERE c.cust_id = a.cust_id;
```

**Simulated new/bad incoming data:**
```sql
INSERT INTO STG_CUSTOMER (CUST_ID, CUST_NAME, OUTSTANDING_AMT, CRID, LOCATION, CUST_CREATED)
VALUES 
('c-102', 'Megan', 3500, 'XYAZ', 'AF', '2023-08-15'),
('c-104', 'Vincet', 5000, 'ABDF', 'TX', '2023-08-15');

INSERT INTO ITEM (CUST_ID, ITEM_ID, ITEM_CATEGORY, ITEM_STATUS, COUNTS, PRICE)
VALUES ('c-104', 'a-104', 'Oil', 'Active', 0, 500);   -- COUNTS = 0 breaks the division
```

**Email notification + alert:**
```sql
CREATE OR REPLACE NOTIFICATION INTEGRATION DT_FAILURE_ALERT
TYPE = EMAIL
ENABLED = TRUE
ALLOWED_RECIPIENTS = ('emailid@gmail.com')
COMMENT = 'Snowflake Dynamic Table Refresh Notification';

CREATE OR REPLACE ALERT DT_FAILURE
WAREHOUSE = COMPUTE_WH
SCHEDULE = '1 MINUTE'
IF (EXISTS (
    SELECT * FROM TABLE(
        INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
            DATA_TIMESTAMP_START => DATEADD('hour', -1, CURRENT_TIMESTAMP()),
            DATA_TIMESTAMP_END   => DATEADD('hour', 0, CURRENT_TIMESTAMP()),
            NAME => 'CUST_ITEM_DT', 
            ERROR_ONLY => TRUE
        )
    )
    ORDER BY name, data_timestamp
))
THEN CALL SYSTEM$SEND_EMAIL(
    'DT_FAILURE_ALERT', 'emailid', 'Dynamic Table failure Notification',
    'Issue with some data, Main Error {Division by Zero}'
);

ALTER ALERT DT_FAILURE RESUME;   -- Alerts are suspended by default, like Tasks
```

**Manual refresh-history check:**
```sql
SELECT * FROM TABLE(
    INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
        DATA_TIMESTAMP_START => DATEADD('hour', -1, CURRENT_TIMESTAMP()),
        DATA_TIMESTAMP_END   => DATEADD('hour', 0, CURRENT_TIMESTAMP()),
        NAME => 'CUST_ITEM_DT', 
        ERROR_ONLY => TRUE
    )
);
```

### Explanation

**Dynamic Tables** automatically re-run their defining query and refresh as source data changes — no Task or manual `MERGE` script needed.

- `QUALIFY ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY CUST_CREATED DESC) = 1` — dedup logic: number rows per customer by recency, keep only rank 1 (newest). `QUALIFY` filters directly on a window function without a wrapping subquery.
- `TARGET_LAG = DOWNSTREAM` — "don't refresh on your own schedule; refresh when a downstream consumer needs fresh data." Used for intermediate tables.
- `INITIALIZE = ON_CREATE` — populate immediately on creation (vs. `ON_SCHEDULE`, which waits for the first scheduled refresh).
- `ITEM_DT` shows the same dedup idea via classic subquery + `WHERE rn = 1` (equivalent to `QUALIFY`).
- `CUST_ITEM_DT` joins the two cleaned tables (old-style comma join + `WHERE`) and computes `price / counts`. It sits at the **top** of the chain, so it needs a real schedule: `TARGET_LAG = '1 MINUTES'`. Since the other two are `DOWNSTREAM`, this top-level schedule drives the whole chain — Snowflake tracks the dependency graph automatically.
- The bad row (`COUNTS = 0`) triggers a **division-by-zero** error on the next refresh — a realistic example of bad data silently breaking a pipeline without active monitoring.
- **Notification Integration** authorizes sending email to a whitelisted recipient set — required before `SYSTEM$SEND_EMAIL` can be used.
- **Alert** = schedule + condition (`IF EXISTS ...`) + action (`THEN CALL ...`). `INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(...)` returns refresh history; `ERROR_ONLY => TRUE` filters to failures only; if any exist in the lookback window, the alert fires the email action.
- `ALTER ALERT ... RESUME` — **Alerts, like Tasks, are suspended by default** and must be explicitly resumed or they never run.

### What Was Demonstrated
1. Built the staging tables + 3-table dynamic chain.
2. Confirmed `CUSTOMER_DT`/`ITEM_DT` deduped correctly.
3. Inserted the `COUNTS = 0` row.
4. Watched `CUST_ITEM_DT` fail with error `1051` (divide by zero) on its next 1-minute refresh.
5. Set up notification + alert, resumed it, received an email within ~1 minute.
6. Fixed the bad row (`UPDATE ITEM SET COUNTS = 5 WHERE ...`) to stop future failures.

### Interview Takeaways

| Concept | Why It Matters |
|---|---|
| Dynamic Tables | Declarative multi-stage transformation pipelines, no manual MERGE scheduling |
| `TARGET_LAG = DOWNSTREAM` | Intermediate tables inherit refresh timing from consumers, avoiding redundant schedules |
| `QUALIFY` | Filter directly on window function results |
| Dynamic Table dependency graph | Snowflake manages refresh order across dependent tables automatically |
| `DYNAMIC_TABLE_REFRESH_HISTORY()` | Table function to audit refresh success/failure |
| Alerts vs. Tasks | Alerts: "check condition → notify," paired with `SYSTEM$SEND_EMAIL`; both suspended by default |
| Notification Integration | Required setup before sending Snowflake emails |

---

## D3. Loading Excel Files via Snowpark (Python)

### Problem
`COPY INTO` handles CSV/JSON/Parquet but **cannot load Excel (.xlsx)** directly. Business teams sending Excel files need another path in: **Snowpark Python** + `pandas` + `openpyxl`, wrapped in a stored procedure so it's callable/schedulable like any other Snowflake object.

### Design
```
Excel file in an internal stage (@EXCEL_DATA)
        │
        ▼
Snowpark Python Procedure:
   1. Download file from stage to local temp storage
   2. Read into a pandas DataFrame
   3. Convert pandas DataFrame → Snowpark DataFrame
   4. Write Snowpark DataFrame into a real Snowflake table
   5. Log outcome (success/failure + row count)
```

### Code
```sql
CREATE OR REPLACE PROCEDURE LOAD_EXCEL_FILES(file_name STRING, target_table STRING) 
RETURNS STRING
LANGUAGE PYTHON
RUNTIME_VERSION = '3.9'
PACKAGES = ('snowflake-snowpark-python', 'pandas', 'openpyxl')
HANDLER = 'main'   
AS
$$
import pandas as pd
from snowflake.snowpark import Session
import traceback 

def main(session: Session, file_name: str, target_table: str) -> str:
    try:
        file_path = f"@EXCEL_DATA/{file_name}"  
        session.file.get(file_path, '/tmp')
        local_path = f"/tmp/{file_name}" 

        panda_df = pd.read_excel(local_path)

        snow_df = session.create_dataframe(panda_df)

        try:
            snow_df.write.save_as_table(target_table, mode="overwrite")

            row_count = len(panda_df)
            log_status(session, file_name, target_table, "SUCCESS", None, row_count)
            return f"{row_count} rows into table '{target_table}' Loaded Successfully"

        except Exception as e:
            log_status(session, file_name, target_table, "FAILED", str(e), 0)
            return f"Failed to write table: {e}"

    except Exception as e:
        log_status(session, file_name, target_table, "FAILED", traceback.format_exc()[:1500], 0)
        return f"Procedure could not execute. Error: {str(e)}"


def log_status(session, file_name, target_table, status, error_message=None, row_count=0):
    try:
        insert_stmt = f"""
            INSERT INTO EXCEL_LOAD_LOGS (
                FILE_NAME, TARGET_TABLE, STATUS, ERROR_MESSAGE, ROW_COUNT, ETL_LOAD_TIME
            )
            VALUES (
                '{file_name}',
                '{target_table}',
                '{status}',
                {'NULL' if error_message is None else "'" + error_message.replace("'", "''") + "'"},
                {row_count},
                CURRENT_TIMESTAMP()
            )
        """
        session.sql(insert_stmt).collect()
    except Exception as log_err:
        print("Log insert failed:", log_err)
$$;
```

### Explanation

**Header:** A Python stored procedure (necessary since Excel parsing needs `pandas`/`openpyxl`). Parameterized (`file_name`, `target_table`) for reuse. `PACKAGES` lists pre-approved Anaconda-hosted packages available in the sandbox. `HANDLER = 'main'` names the entry-point function.

**`main(session, file_name, target_table)`** — every Snowpark Python handler auto-receives a `session` object (live Snowflake connection) as its first arg.

**Download from stage:** `pandas.read_excel()` needs a real local file — Snowpark's sandbox has its own local filesystem (`/tmp`), so `session.file.get(...)` downloads the file there first.

**Read + convert:** `pd.read_excel(local_path)` reads via pandas (the whole reason Python is needed — no SQL equivalent). `session.create_dataframe(panda_df)` converts the in-memory pandas DataFrame into a **Snowpark DataFrame** Snowflake understands.

**Write:** `snow_df.write.save_as_table(target_table, mode="overwrite")` writes it into a real table; `"overwrite"` replaces existing contents (vs. `"append"`).

**Nested try/except:** an **outer** block catches download/read/convert failures; an **inner** block isolates *write*-step failures specifically — letting you distinguish "the Excel file is broken" from "the table write failed" (e.g. schema mismatch/permissions), which need different fixes. `traceback.format_exc()[:1500]` captures the full stack trace, truncated to fit a log column.

**`log_status()` helper:**
- `{'NULL' if error_message is None else "'" + error_message.replace("'", "''") + "'"}` — inserts literal `NULL` if no error, otherwise quotes the message with single quotes escaped (`''`) to prevent broken SQL/injection from apostrophes in error text.
- `session.sql(insert_stmt).collect()` builds and executes the SQL (`.collect()` triggers execution since Snowpark queries are otherwise lazy).
- The logging call is itself wrapped in try/except so a *logging* failure never crashes the whole procedure — it just prints a message.

### Interview Takeaways

| Concept | Why It Matters |
|---|---|
| Python Stored Procedures | Needed when native SQL/`COPY INTO` can't handle a format (Excel) |
| `PACKAGES` clause | Declares sandboxed Python libraries available to the procedure |
| `session.file.get()` | Downloads a stage file into the local Python sandbox filesystem |
| pandas → Snowpark DataFrame | Bridge between regular Python data and Snowflake-native data |
| `save_as_table(mode="overwrite")` | Writes a DataFrame to a real table; overwrite vs. append matters |
| Nested try/except | Isolates *which stage* failed for precise debugging |
| Escaping quotes before dynamic insert | Prevents broken SQL/injection from arbitrary error text |

---

## D4. Auto Table Creation & Load from S3 via Snowpark + Task

### Problem
Multiple teams drop CSVs into a shared S3 bucket, each in its own folder. Requirements: (1) auto-detect new folders/files, (2) auto-create a matching table for new folders, (3) load only new files, never reload processed ones, (4) run on a schedule (Task), (5) auto-email on failure.

### Design
```
S3 Bucket
 ├── customer/customer.csv
 ├── nation/nation.csv
 ├── region/region.csv
 └── supplier/supplier.csv
        │
        ▼
Snowpark Python Procedure (scheduled via TASK):
   1. LIST all files in the external stage
   2. Compare against LOG_TABLE — skip already-processed files
   3. For new files: extract folder name (→ table name) + file name
   4. If table doesn't exist → CREATE TABLE via INFER_SCHEMA
   5. COPY INTO the table
   6. INSERT into LOG_TABLE marking file processed
        │
        ▼
   If the task fails → SNS/email notification fires automatically
```

### Code (Python handler)
```python
import snowflake.snowpark as snowpark
from snowflake.snowpark.functions import col

def main(session: snowpark.Session) -> str:
    stage = "@S3_STAGE"
    file_format = "csv_format"
    created_tables = []
    log_messages = []

    stage_files_df = session.sql(f"LIST {stage}").collect()

    loaded_files_df = session.table("LOG_TABLE").select("FILE_NAME").collect()
    already_loaded_files = {row["FILE_NAME"] for row in loaded_files_df}

    for row in stage_files_df:
        file_path = row['name']  # e.g., s3://bucket/folder/file.csv

        if file_path in already_loaded_files:
            msg = f"Skipping already loaded file: {file_path}"
            print(msg)
            log_messages.append(msg)
            continue

        parts = file_path.split('/')

        if len(parts) > 1:
            folder = parts[-2]
            file_name = parts[-1]
            table_name = folder.upper()

            full_path = f"{stage}/{folder}/{file_name}"

            exists_result = session.sql(f"""
                SELECT COUNT(*) AS count 
                FROM INFORMATION_SCHEMA.TABLES 
                WHERE TABLE_NAME = '{table_name}' 
                AND TABLE_SCHEMA = CURRENT_SCHEMA()
            """).collect()
            
            exists = int(exists_result[0]['COUNT']) if exists_result else 0

            if exists == 0:
                create_table_sql = f"""
                    CREATE OR REPLACE TABLE {table_name}
                    USING TEMPLATE (
                        SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
                        FROM TABLE(
                            INFER_SCHEMA(LOCATION => '{stage}/{folder}/', FILE_FORMAT => '{file_format}')
                        )
                    )
                """
                session.sql(create_table_sql).collect()
                created_tables.append(table_name)
                log_messages.append(f"Created new table: {table_name}")

            copy_sql = f"""
                COPY INTO {table_name}
                FROM {full_path}
                FILE_FORMAT = (FORMAT_NAME = '{file_format}')
                MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
                ON_ERROR = ABORT_STATEMENT
            """
            session.sql(copy_sql).collect()
            log_messages.append(f"Loaded data into table: {table_name} from {file_name}")

            session.sql(f"""
                INSERT INTO LOG_TABLE (FOLDER_NAME, FILE_NAME)
                VALUES ('{folder}', '{file_path}')
            """).collect()
            log_messages.append(f"Logged file: {file_path}")

    if not log_messages:
        log_messages.append("No new files processed. All files were already loaded.")

    message = "\n".join(log_messages)
    print("Final Result:\n", message)

    return session.create_dataframe([[message]], schema=["RESULT"])
```

### Supporting SQL Setup
```sql
CREATE OR REPLACE SCHEMA AUTO;

CREATE OR REPLACE TABLE LOG_TABLE (
    FOLDER_NAME STRING,
    FILE_NAME STRING,
    LOAD_TIMESTAMP TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

CREATE OR REPLACE FILE FORMAT csv_format
  TYPE = 'CSV'
  PARSE_HEADER = TRUE;   -- required for INFER_SCHEMA (not SKIP_HEADER)

CREATE OR REPLACE STORAGE INTEGRATION s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = '<role-arn-from-aws>'
  STORAGE_ALLOWED_LOCATIONS = ('s3://realtime-project-snowflake/');

CREATE OR REPLACE STAGE S3_STAGE
  URL = 's3://realtime-project-snowflake/'
  STORAGE_INTEGRATION = s3_int;

CREATE OR REPLACE NOTIFICATION INTEGRATION task_error_notification
  TYPE = QUEUE
  NOTIFICATION_PROVIDER = AWS_SNS
  ENABLED = TRUE
  AWS_SNS_TOPIC_ARN = '<sns-topic-arn>'
  AWS_SNS_ROLE_ARN = '<same-role-arn>';

CREATE OR REPLACE TASK autoload_tables
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = '1 MINUTE'
  ERROR_INTEGRATION = task_error_notification
AS
  CALL SP_AUTO_S3_TABLES();

ALTER TASK autoload_tables RESUME;  -- Tasks are suspended by default
```

### Explanation

**`LIST @S3_STAGE`** — returns every file currently in the stage (full path, size, last-modified) — this is how new files are "discovered" without prior knowledge.

**Already-loaded lookup:** `{row["FILE_NAME"] for row in loaded_files_df}` builds a Python **set** for O(1) fast membership checks — matters at scale with many files.

**Main loop / idempotency:** for each stage file, if already in the processed set, `continue` — skips reprocessing, preventing duplicate loads across repeated (e.g. every-minute) runs.

**Folder → table name:** splitting `s3://bucket/customer/customer.csv` on `/`, `parts[-1]` = file name, `parts[-2]` = folder name → becomes the (uppercased) table name. **This is the core trick:** S3 folder structure dictates the Snowflake table structure — drop a new folder, get a new table automatically.

**Existence check:** queries `INFORMATION_SCHEMA.TABLES` for a matching `TABLE_NAME`; count `0` = table doesn't exist yet.

**Auto-create via `INFER_SCHEMA`:** scans the file(s) at the folder location, detects columns/types, and `CREATE TABLE ... USING TEMPLATE (...)` builds the table from that inferred structure — no manual DDL needed regardless of what any team's CSV looks like. Requires `PARSE_HEADER = TRUE` (not `SKIP_HEADER`) in the file format so `INFER_SCHEMA` can read column names correctly.

**Load:** standard `COPY INTO` with `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` for safety alongside the auto-inferred schema.

**Log processed file:** feeds next run's `already_loaded_files` set, completing the idempotency loop.

**Return type:** returns a Snowpark DataFrame (`session.create_dataframe([[message]], schema=["RESULT"])`) rather than a plain string — a valid alternative when you want output shaped like a query result.

**Orchestration (Task):** `SCHEDULE = '1 MINUTE'` triggers the procedure automatically; `ERROR_INTEGRATION = task_error_notification` publishes failures to an AWS SNS topic (subscribed → email) if the call throws an unhandled error; `ALTER TASK ... RESUME` is required since **Tasks are suspended by default**.

**AWS-side setup** — two integrations following the same secure-handshake pattern:
1. **Storage Integration** — lets Snowflake read S3 without hardcoded keys, via an IAM role trusting a Snowflake-provided external ID.
2. **Notification Integration (`TYPE = QUEUE`, `AWS_SNS`)** — lets a Task publish failure messages to an SNS topic subscribed to an email address.

Both require: create/identify an IAM role in AWS → get its ARN + external ID from Snowflake's `DESCRIBE INTEGRATION` output → paste into the IAM role's trust policy.

### What Was Demonstrated
1. First task run processed all 5 existing folders/files, auto-creating 5 tables, logging each.
2. Added a new folder (`line_item/`) — next scheduled run auto-detected it, created `LINE_ITEM`, loaded data — zero code changes.
3. Deliberately broke the stage reference (typo) to simulate failure — task failed, SNS email arrived automatically with the real error.

### Interview Takeaways

| Concept | Why It Matters |
|---|---|
| `LIST @stage` | Programmatically discover files in a stage |
| Set-based "already processed" lookup | Efficient idempotency check |
| Folder name → table name convention | Powerful pattern for self-expanding ingestion frameworks |
| `INFORMATION_SCHEMA.TABLES` existence check | Standard conditional-create pattern |
| `INFER_SCHEMA` + `CREATE TABLE ... USING TEMPLATE` | Auto-detect + build table schema from a file, zero manual DDL |
| `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` | Safer loading alongside inferred schemas |
| Snowpark Python for orchestration | Better suited than pure SQL for looping/parsing/branching |
| Task `ERROR_INTEGRATION` vs. Alert-based monitoring | Two valid, different failure-notification mechanisms |
| Storage + Notification Integration (SNS) | Standard two-way trust pattern connecting Snowflake ↔ AWS securely |
| Tasks suspended by default | Common gotcha — always `ALTER TASK ... RESUME` (Alerts too; Snowpipes, by contrast, start automatically) |

---

# Part E — Cross-Cutting

## E1. Patterns That Repeat Across All Scenarios

1. **Validate-before-commit staging** (D1): load into temp/staging first, check quality, only then commit to the real table.
2. **Always log the outcome, success or failure** (all scenarios): audit tables are non-negotiable in production pipelines.
3. **Nested error handling** (D1, D3): isolate *which step* failed (load vs. validate vs. write) for actionable, specific error messages.
4. **Suspended-by-default objects**: Tasks and Alerts both start suspended and must be explicitly resumed — Snowpipes, by contrast, start running immediately. Always verify object state after creation.
5. **Automated failure notification**, two common approaches:
   - **Alerts** + `SYSTEM$SEND_EMAIL` + a `TYPE = EMAIL` Notification Integration — good for monitoring conditions on data/objects like Dynamic Tables.
   - **Task `ERROR_INTEGRATION`** + AWS SNS + a `TYPE = QUEUE` Notification Integration — good for monitoring scheduled task execution failures.
6. **Dynamic/parameterized object names** via `IDENTIFIER(:variable)` (SQL) or f-strings (Python) — makes procedures reusable instead of hardcoded to one table.
7. **Schema-on-read automation** via `INFER_SCHEMA` — removes manual DDL when source files are unpredictable or numerous.
8. **Snowpark Python bridges the gap** wherever pure SQL can't: reading non-native formats (Excel), complex looping/branching, dynamic multi-table orchestration.

---

## E2. Validation Notes on This Consolidation

The source content was technically sound overall (standard, well-known Snowflake features and syntax). Changes made while organizing:

- Restructured everything under one table of contents with consistent section numbering (A–E) so the two original documents (Stages notes + two scenario sets) read as a single reference instead of three disconnected pastes.
- Removed duplicated boilerplate (e.g., the Stage doc's core content and the "simplified recap" version were merged into one section with the recap kept as a condensed callout, rather than repeating the full explanation twice).
- The Iceberg Tables item was only a bare link in your source with no notes — flagged clearly as a pointer/stub in Part B rather than fabricating content, and I did not reproduce text from the linked Medium article; only a short, original-wording summary of its general comparison angle is given, per copyright limits on quoting external sources.
- Left all SQL/Python exactly as provided (not independently execution-tested) — a few syntax notes worth flagging if you plan to run this as-is:
  - `RETURNS VARCHAR()` and `error_message VARCHAR()` (empty parens) in Scenario D1 aren't standard Snowflake Scripting syntax — typically you'd write `RETURNS VARCHAR` / `VARCHAR(16777216)` or a specific length.
  - In C1/C4/C5, `DECLARE` blocks mix Snowflake Scripting styles (e.g., `LET x := ...` appearing *inside* a `BEGIN` block that already has a `DECLARE` section) — both are valid Snowflake Scripting patterns, but usually you pick one style consistently.
  - `CREATE OR ALTER STAGE` (Scenario C4) — double check this exact syntax against current Snowflake docs; `CREATE STAGE ... ` or `ALTER STAGE ...` are the standard forms, and `CREATE OR ALTER` support varies by object type and Snowflake version.
