# Snowflake: Internal Stage vs External Stage

## Simple Explanation

Think of a **stage** as a "waiting area" where you keep files before loading them into a Snowflake table (or after unloading data out of a table).

- **Internal Stage** = Storage space *inside* Snowflake. Snowflake manages it completely — you don't need any cloud account of your own.
- **External Stage** = A pointer to a bucket in *your own* cloud storage (AWS S3, Azure Blob, or Google Cloud Storage). Snowflake just "looks into" this bucket; it doesn't own the storage.

Simple analogy:
- Internal stage = a locker inside Snowflake's building. Only Snowflake has the key.
- External stage = your own storage unit outside. Snowflake has a key too, but so can other tools/people.

---

## Difference Table

| Feature | Internal Stage | External Stage |
|---|---|---|
| **Storage location** | Inside Snowflake | Your own S3 / Azure Blob / GCS bucket |
| **Who manages it** | Snowflake | You (the cloud account owner) |
| **Setup complexity** | Simple, no extra setup | Needs storage integration / credentials + bucket permissions |
| **Cost** | Part of Snowflake storage billing | Billed separately by your cloud provider |
| **Access outside Snowflake** | Not possible — only via Snowflake | Yes — other tools (Spark, other DBs, apps) can access same files |
| **Types** | User stage (`@~`), Table stage (`@%table`), Named internal stage | Named external stage pointing to a cloud URL |
| **Data ownership/compliance control** | Limited — Snowflake controls it | Full control — useful for compliance/data residency |
| **Best for** | Quick, simple, one-off loading/unloading | Data lake architecture, shared access, large recurring pipelines |

---

## When to Use Which

### Use Internal Stage when:
- You don't have (or don't want to manage) your own cloud storage
- You just need a quick place to stage files before `COPY INTO`
- Nothing else outside Snowflake needs to read those files
- You want the simplest possible setup

### Use External Stage when:
- You already have a data lake (S3/ADLS/GCS) as part of your pipeline
- Multiple tools (not just Snowflake) need to read the same raw files
- You need control over storage cost, retention, region, or encryption keys
- You're doing large-scale, frequent, recurring data loads
- Compliance requires you to control exactly where data physically lives

### Rule of Thumb
> If Snowflake is your **only** consumer of the data → **Internal Stage**.
> If the data is shared across systems or is part of a bigger data lake → **External Stage**.

---

## Interview Questions & Answers

**Q1. What is a stage in Snowflake?**
A stage is a location (internal or external) where data files are stored temporarily before being loaded into Snowflake tables, or after being unloaded from tables.

**Q2. What are the types of internal stages?**
- User stage (`@~`) — automatically created for every user
- Table stage (`@%table_name`) — automatically created for every table
- Named internal stage — created manually with `CREATE STAGE`

**Q3. What is the main difference between internal and external stage?**
Internal stage storage is managed by Snowflake itself; external stage storage lives in the customer's own cloud storage (S3/Azure/GCS), and Snowflake only references it.

**Q4. Can you access files in an internal stage from outside Snowflake?**
No. Files in an internal stage can only be accessed through Snowflake (SQL commands, SnowSQL, drivers, GET/PUT commands).

**Q5. What is a storage integration, and why is it needed for external stages?**
A storage integration is a Snowflake object that stores a generated identity (IAM role/service principal) used to securely authenticate to your cloud storage — so you don't have to hard-code cloud credentials in the stage definition.

**Q6. Which is cheaper — internal or external stage?**
It depends on scale. Internal stage costs are bundled into Snowflake's storage pricing, while external stage costs are billed directly by the cloud provider — often cheaper at large scale since you control storage tiers, lifecycle rules, etc.

**Q7. Can the same external bucket be used by other tools besides Snowflake?**
Yes — that's one of the main advantages. Since it's your own cloud bucket, other systems like Spark, other databases, or applications can access the same files.

**Q8. How do you load data into Snowflake using a stage?**
```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (TYPE = 'CSV');
```

**Q9. What commands are used with internal stages that aren't used with external stages?**
`PUT` (upload a local file to an internal stage) and `GET` (download a file from an internal stage to local disk). External stages don't use PUT/GET since files are managed directly in the cloud provider's console/tools.

**Q10. Why would a company choose an external stage over internal despite the extra setup?**
For compliance/data residency control, integration with an existing data lake, sharing files across multiple tools, and potentially lower cost at scale.

**Q11. Is data automatically encrypted in internal stages?**
Yes, Snowflake automatically encrypts data at rest in internal stages using Snowflake-managed keys.

**Q12. Can you list files in a stage?**
Yes:
```sql
LIST @my_stage;
```
snowflake iceberg tables

https://medium.com/snowflake/snowflake-managed-vs-self-managed-iceberg-tables-what-actually-determines-the-difference-c5615ba0d280


Below scenarios from Vishal Kuashal videos

# Snowflake Real-Time Scenarios – Simple Explanations with Full Code Walkthrough

This guide covers **5 real-world Snowflake scenarios** that are commonly asked in interviews and faced in actual data engineering projects. Each scenario explains **the problem, the idea behind the solution, and a line-by-line explanation of the code** in simple language.

---

## Table of Contents
1. [Automate Data Loads from S3 Dynamic Day-Wise Folders](#scenario-1)
2. [Load Valid Data & Auto-Send Failure Records to Client](#scenario-2)
3. [Schema Evolution in Snowflake](#scenario-3)
4. [Dynamic Ingestion from Multiple S3 Folders to Multiple Tables](#scenario-4)
5. [Dynamic Table Creation from CSV File Headers](#scenario-5)
6. [Track DDL Changes in Snowflake (Audit Logging)](#scenario-6)

---

<a name="scenario-1"></a>
## 1. Automate Data Loads from S3 Dynamic Day-Wise Folders to Snowflake

### 🧩 The Problem
A client stores files in AWS S3 using a **folder structure organized by date**, like this:

```
snow_aws_project / 2025 / 06 / 20 / customer.csv
```

Every day a **new folder** gets created (year → month → day), and inside it there is one file that needs to be loaded into Snowflake. Since the date changes daily, you can't hardcode the path — you need it to be **built automatically** based on the current date.

This is a very common interview question because in real projects you rarely load static file paths — folders change every day.

### 💡 The Idea
1. Extract the **year, month, and day** from the current date.
2. **Concatenate** these parts to build the S3 folder path dynamically (e.g. `2025/06/20/`).
3. Use this dynamic path inside a `COPY INTO` command to load the file.
4. Wrap all of this inside a **stored procedure** so it can run automatically (and later be scheduled with a Task).

### 🛠️ Code Explanation

**Step 1 – Create the target table** (schema copied from Snowflake's sample data, but empty):
```sql
CREATE OR REPLACE TABLE CUSTOMER  AS
SELECT * FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF1.CUSTOMER where 1=2;
```
`WHERE 1=2` is a trick to copy only the **table structure**, not the actual rows.

**Step 2 – Prerequisite objects** (created earlier in the series, not repeated here):
- A **File Format** (defines the file is CSV)
- A **Storage Integration** (the secure connection between Snowflake and AWS S3)
- A **Stage** pointing to the *top-level* S3 folder only (no year/month/day in the stage URL — this keeps it generic so it works for every future year automatically)

**Step 3 – The Stored Procedure** (conceptual structure used in the video):
```sql
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
    -- Step 1: Extract year, month, day from today's date
    yr := TO_CHAR(CURRENT_DATE, 'YYYY');
    mn := TO_CHAR(CURRENT_DATE, 'MM');
    dy := TO_CHAR(CURRENT_DATE, 'DD');

    -- Step 2: Build the dynamic S3 folder path
    full_path := yr || '/' || mn || '/' || dy || '/';

    -- Step 3: Build the COPY INTO statement as a string
    copy_sql := 'COPY INTO CUSTOMER FROM @DYNAMIC_STAGE/' || full_path;

    -- Step 4: Run it immediately
    EXECUTE IMMEDIATE :copy_sql;

    RETURN copy_sql;
END;
$$;

CALL DYNAMIC_LOAD();
```

**What each part means, in plain English:**
| Code Part | What it does |
|---|---|
| `DECLARE ... yr, mn, dy` | Creates temporary "boxes" (variables) to hold the year, month, and day |
| `TO_CHAR(CURRENT_DATE, 'YYYY')` | Pulls just the year part from today's date, e.g. `2025` |
| `full_path := yr \|\| '/' \|\| mn \|\| '/' \|\| dy \|\| '/'` | Joins (concatenates) year, month, day with `/` in between to form a folder path like `2025/06/20/` |
| `copy_sql := 'COPY INTO ...' \|\| full_path` | Builds the actual SQL command **as text**, since the path is not known until the procedure runs |
| `EXECUTE IMMEDIATE :copy_sql` | Tells Snowflake: "Take this text and actually run it as a real SQL command" (this is called **Dynamic SQL**) |
| `RETURN copy_sql` | Returns the final SQL text so you can see exactly what got executed — useful for **audit/debugging** |

### ⚠️ Important Gotcha
If you `CALL` the procedure twice, the second run **won't insert anything new** — because the same file was already loaded and `COPY INTO` skips files it has already processed. To reload, either:
- `TRUNCATE TABLE CUSTOMER` first, or
- add `FORCE = TRUE` to the `COPY INTO` command.

### 🎯 Bonus Interview Variation
Sometimes instead of folders, the client gives one file per day in the **same folder**, named like `customer_20250621.csv`. In that case you don't build a folder path — you build a **file name pattern** and pass it to `COPY INTO ... PATTERN = '...'`, using the same year/month/day extraction logic.

---

<a name="scenario-2"></a>
## 2. Load Valid Data & Auto-Send Failure Records Back to Client

### 🧩 The Problem
The client drops CSV files into S3. Some rows are **good** (match the table structure) and some rows are **bad** (wrong data type, extra/missing columns, etc.). The requirement:
1. Load all **correct** rows into the main table.
2. Put all **failed/rejected** rows into a separate table.
3. **Export** those failed rows back to a different S3 folder so the client can see and fix them.

### 💡 The Idea
Snowflake's `COPY INTO` command has an `ON_ERROR` option. Setting it to `CONTINUE` means: *"skip bad rows, but still load the good ones."* Then Snowflake's special `VALIDATE()` table function can tell you exactly **which rows failed and why** from the last `COPY INTO` run. Finally, another `COPY INTO` (in the reverse direction — table to S3) sends the bad rows back out.

### 🛠️ Code Explanation

**Step 1 – Create table, file format, storage integration, and two stages** (one for reading source files, one for writing rejected files):
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

**Step 2 – Load only the good rows, skipping bad ones:**
```sql
COPY INTO CUSTOMER
FROM @AWS_STAGE
ON_ERROR = CONTINUE;
```
`ON_ERROR = CONTINUE` means: if a row fails (e.g., wrong number of columns), **skip it and move to the next row** instead of stopping the whole load.

**Step 3 – Find out exactly which rows failed:**
```sql
SELECT * FROM TABLE(VALIDATE(CUSTOMER, JOB_ID => '_last'));
```
`VALIDATE()` is a built-in Snowflake function that looks back at the **most recent** `COPY INTO` run (`JOB_ID => '_last'`) on that table and lists every row that was rejected, along with the error reason, file name, and row number.

**Step 4 – Save those failed rows into their own table:**
```sql
CREATE OR REPLACE TABLE FAULTY_RECORDS AS
SELECT FILE, CATEGORY, ERROR, REJECTED_RECORD
FROM TABLE(VALIDATE(CUSTOMER, JOB_ID => '_last'));
```

**Step 5 – Send the faulty records back out to S3:**
```sql
COPY INTO @AWS_TARGET_STAGE/faultyrecords.csv
FROM FAULTY_RECORDS
SINGLE = TRUE;
```
`SINGLE = TRUE` forces Snowflake to write **one single file** instead of splitting the output into multiple small files (which it normally does for performance).

### 🛠️ Making It Reusable: The Stored Procedure

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
    -- Step 1: Load valid records into the target table
    LET COPY_SQL := '
        COPY INTO ' || TABLE_NAME || '
        FROM @' || SOURCE_STAGE || '
        ON_ERROR = CONTINUE;
    ';
    EXECUTE IMMEDIATE :COPY_SQL;

    -- Step 2: Save failed records into a new table
    LET VALIDATE_SQL := '
        CREATE OR REPLACE TABLE FAULTY_RECORDS AS
        SELECT FILE, CATEGORY, ERROR, REJECTED_RECORD
        FROM TABLE(
            VALIDATE(' || TABLE_NAME || ', JOB_ID => ''_last'')
        );
    ';
    EXECUTE IMMEDIATE :VALIDATE_SQL;

    -- Step 3: Export failed records to S3 stage
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

**Why parameters (`TABLE_NAME`, `SOURCE_STAGE`, `TARGET_STAGE`)?**
So the **same procedure** can be reused for the Product table, Order table, Inventory table, etc. — instead of writing one procedure per table.

**Why 3 separate `EXECUTE IMMEDIATE` blocks?**
Each SQL step (load → validate/save → export) is built as **text first** (because table/stage names are dynamic parameters, not fixed), then executed one after another. This is standard practice for writing **generic, reusable stored procedures**.

### 📝 Automating Further
Instead of manually calling this procedure every time the client uploads a file, you can wrap it inside a **Snowflake Task** and schedule it to run at fixed times (e.g. 12 PM, 5 PM, 8 PM) — fully automated, no manual work.

---

<a name="scenario-3"></a>
## 3. Schema Evolution in Snowflake

### 🧩 The Problem
You built a pipeline that loads a client's CSV file into a table. Then one day the client **adds a new column** to their file (or reorders columns). Normally this **breaks your `COPY INTO` load** because the file no longer matches the table structure — forcing manual fixes every time.

### 💡 The Idea
Snowflake has two related features:
1. **Schema Detection** — Snowflake can look at a file and automatically figure out its column names and data types (no need to manually define the table).
2. **Schema Evolution** — Once a table exists, Snowflake can **automatically add new columns** to that table when a differently-shaped file is loaded — no manual `ALTER TABLE` required.

**Important limitation:** this feature only works with `COPY INTO` (or Snowpipe) — **not** with plain `INSERT` statements.

### 🛠️ Code Explanation

**Step 1 – File format must use `PARSE_HEADER = TRUE`** (not `SKIP_HEADER`), because schema detection needs to read the actual column names from the header row:
```sql
CREATE OR REPLACE FILE FORMAT CSVTYPE
TYPE='CSV'
PARSE_HEADER=TRUE
RECORD_DELIMITER='\n'
FIELD_DELIMITER=','
FIELD_OPTIONALLY_ENCLOSED_BY='"'
ENCODING='UTF8';
```

**Step 2 – Storage integration + stage** (standard setup, same as before):
```sql
CREATE OR REPLACE STORAGE INTEGRATION S3_INT ...;
CREATE OR REPLACE STAGE SCHEMA_DETECT ...;
LIST @SCHEMA_DETECT;
```

**Step 3 – Detect the schema automatically from a file:**
```sql
SELECT * FROM TABLE(
    INFER_SCHEMA(
        LOCATION => '@SCHEMA_DETECT/customer_file.csv',
        FILE_FORMAT => 'CSVTYPE',
        IGNORE_CASE => TRUE
    )
);
```
`INFER_SCHEMA()` reads the file and returns a list of column names + their guessed data types. `IGNORE_CASE => TRUE` means column name casing (upper/lower) won't matter.

**Step 4 – Auto-create a table using that inferred schema:**
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
`USING TEMPLATE` is a special Snowflake syntax that says: *"Build this table's columns from the JSON-like structure I'm feeding you."* `ARRAY_AGG(OBJECT_CONSTRUCT(*))` simply packages all the inferred column definitions into one array object that `CREATE TABLE ... USING TEMPLATE` understands.

**Step 5 – Load the first file (matches table exactly, 8 columns):**
```sql
COPY INTO CUSTOMER_DATA
FROM @SCHEMA_DETECT/customer_us_file.csv
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```
`MATCH_BY_COLUMN_NAME` tells Snowflake to line up file columns to table columns **by name**, not by position — so even if columns are reordered, it still works correctly. `CASE_INSENSITIVE` ignores upper/lower case differences.

**Step 6 – Enable Schema Evolution before loading a file with EXTRA columns (9 columns instead of 8):**
```sql
ALTER TABLE CUSTOMER_DATA SET ENABLE_SCHEMA_EVOLUTION = TRUE;
ALTER FILE FORMAT CSVTYPE SET ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE;
```
- `ENABLE_SCHEMA_EVOLUTION = TRUE` → tells the **table** it's allowed to gain new columns automatically.
- `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` → tells the **file format** to not throw an error just because the file has a different number of columns than the table.

**Step 7 – Load the new file with the extra column:**
```sql
COPY INTO CUSTOMER_DATA
FROM @SCHEMA_DETECT/customer_us_file.csv
MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```
This time, since the file has an extra column (e.g., `AUDIT_DATE`) that the table doesn't have, Snowflake **automatically adds that column to the table** and loads the data. Old rows that didn't have this column will simply show `NULL` for it.

### ✅ Key Takeaways
| Setting | Purpose |
|---|---|
| `INFER_SCHEMA` | Auto-detects column names/types from a file |
| `USING TEMPLATE` | Auto-creates a table from detected schema |
| `MATCH_BY_COLUMN_NAME` | Matches file columns to table columns by name (handles reordering) |
| `ENABLE_SCHEMA_EVOLUTION` | Lets the table auto-add new columns |
| `ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE` | Stops Snowflake from rejecting files with a different column count |

---

<a name="scenario-4"></a>
## 4. Dynamic Ingestion from Multiple S3 Folders to Multiple Tables

### 🧩 The Problem
An S3 bucket has **multiple folders** (e.g., `customer/`, `nation/`, `region/`, `orders/`), and each folder's files must go into a **matching table** with the same name. Tomorrow, the client might add a brand-new folder (e.g., `supplier/`) — and the pipeline must **automatically** create a new table for it and load its data, **without any code changes**.

### 💡 The Idea
1. Enable a **Directory Table** on the stage — this gives Snowflake a queryable list of every file path in the bucket (`RELATIVE_PATH` column).
2. Loop through every file path using a **CURSOR**.
3. From each path, extract the **folder name** (this becomes the table name) and the **file name**.
4. Check an **audit table** to see if this exact file was already loaded before (to avoid loading it twice).
5. If the table for that folder doesn't exist yet → create it automatically using `INFER_SCHEMA` (like Scenario 3).
6. Load the file into the table using `COPY INTO ... MATCH_BY_COLUMN_NAME`.
7. Log the file as "processed" in the audit table.

### 🛠️ Code Explanation

**Step 1 – Setup: schema, file format, storage integration:**
```sql
CREATE OR REPLACE SCHEMA BRONZE;
CREATE OR REPLACE FILE FORMAT CSVONE CLONE RAWLAYER.CSVTYPE;
CREATE OR REPLACE STORAGE INTEGRATION DYNAMIC_INT ...;
```

**Step 2 – Create a stage with a Directory Table enabled:**
```sql
CREATE OR ALTER STAGE DYNAMICLOAD
FILE_FORMAT = CSVONE
STORAGE_INTEGRATION = DYNAMIC_INT
URL = 's3://realtimeproject-snowflake/'
DIRECTORY = (ENABLE = true, AUTO_REFRESH = TRUE);
```
`DIRECTORY = (ENABLE = true)` turns on a special hidden table that keeps track of every file's exact path in that S3 bucket. `AUTO_REFRESH = TRUE` means this list updates itself automatically whenever new files arrive (using an S3 event notification behind the scenes).

**Step 3 – Query the directory table to see all file paths:**
```sql
SELECT RELATIVE_PATH FROM DIRECTORY(@DYNAMICLOAD);
```
This returns rows like:
```
customer/customer.csv
nation/nation.csv
region/region.csv
```
The part **before the slash** is the folder name (→ becomes the table name). The part **after the slash** is the file name.

**Step 4 – Create an audit table to avoid reprocessing the same file:**
```sql
CREATE OR REPLACE TABLE FILE_LOAD_LOG (
    FILE_NAME STRING,
    LOAD_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Step 5 – The full stored procedure:**
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

        -- Skip if this file was already loaded before
        SELECT COUNT(1) INTO :record_count FROM FILE_LOAD_LOG WHERE FILE_NAME = :file_path;
        IF (record_count > 0) THEN
            CONTINUE;
        END IF;

        tablename := UPPER(folder_name);
        full_stage_path := '@DYNAMICLOAD/' || file_path;

        -- Check if the table already exists
        SELECT COUNT(1) INTO :record_count
        FROM INFORMATION_SCHEMA.TABLES
        WHERE TABLE_SCHEMA = CURRENT_SCHEMA()
          AND TABLE_NAME = :tablename;

        IF (record_count = 0) THEN
            -- Table doesn't exist yet — create it using inferred schema
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

        -- Load the file into the (existing or newly created) table
        LET copy_sql := '
            COPY INTO IDENTIFIER(?)
            FROM ?
            FILE_FORMAT = (FORMAT_NAME = ''CSVONE'')
            MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
            ON_ERROR = ''SKIP_FILE''';
        EXECUTE IMMEDIATE copy_sql USING (tablename, full_stage_path);

        -- Record this file as processed
        INSERT INTO FILE_LOAD_LOG (FILE_NAME) VALUES (:file_path);
    END FOR;

    RETURN 'Dynamic ingestion from S3 to Snowflake complete.';
END;
$$;

CALL DYNAMIC_TABLE_LOAD();
```

**Breaking down the tricky parts:**

| Code Part | Plain-English Meaning |
|---|---|
| `CURSOR FOR SELECT RELATIVE_PATH FROM DIRECTORY(@DYNAMICLOAD)` | Prepares a "list" of every file path in the bucket, so we can go through them one at a time |
| `FOR file_rec IN file_cursor DO ... END FOR` | A loop — repeat the steps inside for **every single file path** found |
| `SPLIT_PART(file_path, '/', 1)` | Splits the path by `/` and grabs the **1st piece** (the folder name) |
| `SPLIT_PART(file_path, '/', 2)` | Grabs the **2nd piece** (the file name) |
| `IF (record_count > 0) THEN CONTINUE; END IF;` | If this file is already in the audit log, **skip it** and move to the next file in the loop (avoids duplicate loads) |
| `IDENTIFIER(?)` with `USING (tablename, ...)` | A safe way to plug a **variable table name** into a SQL statement (like a fill-in-the-blank), instead of unsafely gluing strings together |
| `ON_ERROR = 'SKIP_FILE'` | If an entire file fails to load, skip that file (but don't stop the whole procedure) |
| `INSERT INTO FILE_LOAD_LOG` | Marks the file as "done" so it's never reloaded again |

### ✅ Why This Is Powerful
- Add a brand-new folder (e.g. `supplier/`) to S3 → next time the procedure runs, it **automatically creates a `SUPPLIER` table and loads the data** — zero code changes needed.
- The audit table (`FILE_LOAD_LOG`) prevents duplicate loading if the procedure is run again.
- `DIRECTORY(...) REFRESH` / `AUTO_REFRESH = TRUE` + an **S3 Event Notification** keeps the directory table always up to date with new files.

---

<a name="scenario-5"></a>
## 5. Dynamic Table Creation from CSV File Headers (Single-Cell Header Row)

### 🧩 The Problem
Instead of a normal CSV (where each column has its own cell), the client sends a file where **all column names are crammed into a single cell** in the header row, separated by a delimiter like a pipe (`|`):

```
customer_id|customer_name|age|phone_number|address|email
1001,John,25,9999999999,Delhi,john@test.com
```

`INFER_SCHEMA` **cannot** handle this format because it expects properly separated columns. So you must **manually parse the header text**, split it into column names, and build the `CREATE TABLE` statement yourself.

### 💡 The Idea
1. Get the file name from the stage's directory table → use it as the table name.
2. Read just the **header line** (`$1`, the first "column" containing all the text) into a temporary table.
3. **Split** that header text by the pipe delimiter → this becomes an **array** of column names.
4. Find the **array size** (i.e., how many columns exist).
5. **Loop** through the array, building a column-definition string (`"col1" STRING, "col2" STRING, ...`).
6. Dynamically build and execute a `CREATE TABLE` statement with those columns.
7. Load the actual data with `COPY INTO`, skipping the header row this time.

### 🛠️ Code Explanation

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
    -- Step 1: Get the file name from the @HEADER stage's directory table
    SELECT RELATIVE_PATH INTO file_name FROM DIRECTORY(@HEADER);

    -- Step 2: Derive the table name from the file name
    --  e.g. 'customerheader.csv' -> 'CUSTOMERHEADER'
    table_name := UPPER(STRTOK(file_name, '.', 1));

    -- Step 3: Read the header line (all column names, in one cell) into a temp table
    EXECUTE IMMEDIATE '
        CREATE OR REPLACE TEMP TABLE TMP_HEADER_LINE AS
        SELECT $1 AS HEADER_LINE
        FROM @HEADER/' || file_name || '
        (FILE_FORMAT => (CSVTYPE))
        LIMIT 1
    ';

    -- Step 4: Pull that header text into a variable
    SELECT HEADER_LINE INTO header_line FROM TMP_HEADER_LINE;

    -- Step 5: Split the header text by "|" into individual column names
    column_list := SPLIT(header_line, '|');
    column_count := ARRAY_SIZE(column_list);

    -- Step 6: Loop through the array to build the column-definitions string
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

    -- Step 7: Create the table dynamically
    create_table_sql := 'CREATE OR REPLACE TABLE "' || table_name || '" (' || column_definitions || ')';
    EXECUTE IMMEDIATE create_table_sql;

    -- Step 8: Load the actual data (this time skipping the header row)
    copy_into_sql := 'COPY INTO "' || table_name || '" FROM @HEADER/' || file_name ||
                      ' FILE_FORMAT = (TYPE = CSV, FIELD_DELIMITER = '','', SKIP_HEADER = 1)';
    EXECUTE IMMEDIATE copy_into_sql;

    RETURN 'Table "' || table_name || '" created successfully and data also get loaded successfully from file "' || file_name || '".';
END;
$$;

CALL CREATE_DYNAMIC_TABLE_FROM_HEADER();
```

**Line-by-line, in plain English:**

| Code Part | What It Does |
|---|---|
| `SELECT RELATIVE_PATH INTO file_name FROM DIRECTORY(@HEADER)` | Grabs the file's path/name from the stage's directory listing |
| `STRTOK(file_name, '.', 1)` | Splits the file name by the `.` and takes the first piece — so `customerheader.csv` becomes `customerheader` (removes the `.csv` extension) |
| `SELECT $1 AS HEADER_LINE FROM @HEADER/... (FILE_FORMAT => (CSVTYPE)) LIMIT 1` | Reads the file with `$1` meaning "give me the entire first column/cell as one text value" — since the whole header is jammed into a single cell, `$1` captures it all as raw text |
| `SPLIT(header_line, '|')` | Breaks that one long text string into a proper **array** using `\|` as the separator — e.g. `["customer_id", "customer_name", "age", ...]` |
| `ARRAY_SIZE(column_list)` | Counts how many items are in that array (i.e., how many columns the file has) |
| `WHILE (i < column_count) DO ... END WHILE` | A loop that runs once per column, starting at index `0` and stopping just before `column_count` (arrays are 0-indexed, so this covers all elements exactly) |
| `TRIM(column_list[i])` | Removes any accidental leading/trailing spaces around a column name |
| `col_def := '"' || col_name || '" STRING'` | Wraps the column name in double quotes and marks its type as `STRING` — e.g. `"customer_id" STRING` |
| `IF (i > 0) THEN column_definitions := column_definitions \|\| ', '; END IF;` | Adds a comma **before** appending each new column (except the very first one) so the final string is valid SQL, e.g. `"col1" STRING, "col2" STRING` |
| `CREATE OR REPLACE TABLE "..." (...)` | Builds the final `CREATE TABLE` statement using the column-definitions string we just built |
| `COPY INTO ... SKIP_HEADER = 1` | This time we skip the header row on load, since we already extracted the column names separately — we only want the actual data rows now |

### ✅ Key Takeaway
This scenario is really about **manual string parsing + dynamic SQL** when Snowflake's built-in schema detection (`INFER_SCHEMA`) can't understand a non-standard file layout.

---

<a name="scenario-6"></a>
## 6. How to Track DDL Changes in Snowflake (Full Audit Logging)

### 🧩 The Problem
Snowflake makes it easy to track **DML changes** (INSERT/UPDATE/DELETE) using Streams. But there's **no built-in feature** to track **DDL/structural changes** — like a teammate adding a new column, dropping a column, or changing a column's data type. Management wants a full audit trail: *what changed, and when.*

### 💡 The Idea
This works like **comparing two photographs — a "before" picture and an "after" picture**:
1. Keep a **snapshot table** that stores the current list of all tables/columns/data types (the "before" picture).
2. Whenever the procedure runs, compare this snapshot against Snowflake's live `INFORMATION_SCHEMA.COLUMNS` (the "after" picture — the real, current state).
3. Any column that's in the "after" picture but not the "before" picture = **ADDED**.
4. Any column that was in the "before" picture but is missing from "after" = **DROPPED**.
5. Any column present in both, but with a different data type = **MODIFIED**.
6. Log all differences into an audit table, then **refresh the snapshot** so next time's comparison starts fresh.

### 🛠️ Code Explanation

**Step 1 – Take the initial "before" snapshot:**
```sql
CREATE OR REPLACE TABLE CURRENT_SNAPSHOT AS
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_CATALOG = 'YOUTUBELEARNING' AND TABLE_SCHEMA = 'BRONZE';
```
This captures the current state (table name, column name, data type) for every column in the `BRONZE` schema — this is our reference point to compare future changes against.

**Step 2 – Create the audit/log table:**
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

**Step 3 – The stored procedure that detects and logs every change:**
```sql
CREATE OR REPLACE PROCEDURE SP_TRACK_DDL_CHANGES()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN

    -- Step 1: Detect ADDED columns
    -- (columns that exist NOW in INFORMATION_SCHEMA but were NOT in our snapshot)
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        c.TABLE_NAME,
        c.COLUMN_NAME,
        'ADDED' AS CHANGE_TYPE,
        NULL AS OLD_DATA_TYPE,
        c.DATA_TYPE AS NEW_DATA_TYPE,
        CURRENT_TIMESTAMP(),
        CURRENT_USER()
    FROM INFORMATION_SCHEMA.COLUMNS c
    LEFT JOIN CURRENT_SNAPSHOT s
        ON c.TABLE_NAME = s.TABLE_NAME
        AND c.COLUMN_NAME = s.COLUMN_NAME
    WHERE c.TABLE_CATALOG = 'YOUTUBELEARNING'
      AND c.TABLE_SCHEMA = 'BRONZE'
      AND s.COLUMN_NAME IS NULL;   -- no match found in snapshot = new column

    -- Step 2: Detect DROPPED columns
    -- (columns that WERE in our snapshot but are NO LONGER in INFORMATION_SCHEMA)
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        s.TABLE_NAME,
        s.COLUMN_NAME,
        'DROPPED' AS CHANGE_TYPE,
        s.DATA_TYPE AS OLD_DATA_TYPE,
        NULL AS NEW_DATA_TYPE,
        CURRENT_TIMESTAMP(),
        CURRENT_USER()
    FROM CURRENT_SNAPSHOT s
    LEFT JOIN INFORMATION_SCHEMA.COLUMNS c
        ON s.TABLE_NAME = c.TABLE_NAME
        AND s.COLUMN_NAME = c.COLUMN_NAME
        AND c.TABLE_CATALOG = 'YOUTUBELEARNING'
        AND c.TABLE_SCHEMA = 'BRONZE'
    WHERE c.COLUMN_NAME IS NULL;   -- no match found in live schema = column was dropped

    -- Step 3: Detect MODIFIED columns (data type changed)
    INSERT INTO DDL_CHANGE_LOG
    SELECT
        c.TABLE_NAME,
        c.COLUMN_NAME,
        'MODIFIED' AS CHANGE_TYPE,
        s.DATA_TYPE AS OLD_DATA_TYPE,
        c.DATA_TYPE AS NEW_DATA_TYPE,
        CURRENT_TIMESTAMP(),
        CURRENT_USER()
    FROM INFORMATION_SCHEMA.COLUMNS c
    JOIN CURRENT_SNAPSHOT s
        ON c.TABLE_NAME = s.TABLE_NAME
        AND c.COLUMN_NAME = s.COLUMN_NAME
    WHERE c.TABLE_CATALOG = 'YOUTUBELEARNING'
      AND c.TABLE_SCHEMA = 'BRONZE'
      AND c.DATA_TYPE <> s.DATA_TYPE;   -- same column name, different type

    -- Step 4: Refresh the snapshot so it matches the current live state
    DELETE FROM CURRENT_SNAPSHOT;

    INSERT INTO CURRENT_SNAPSHOT
    SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_CATALOG = 'YOUTUBELEARNING'
      AND TABLE_SCHEMA = 'BRONZE';

    RETURN 'DDL change detection completed successfully.';
END;
$$;

CALL SP_TRACK_DDL_CHANGES();
```

**Explaining the join logic (the trickiest part), in plain English:**

| Detection | Join Type Used | Why |
|---|---|---|
| **ADDED** | `INFORMATION_SCHEMA.COLUMNS` (live) `LEFT JOIN` `CURRENT_SNAPSHOT` (old) | We start from the **live/current** data. A `LEFT JOIN` keeps every live column even if there's no match in the old snapshot. If `s.COLUMN_NAME IS NULL`, it means this column exists now but **didn't exist before** → it was just added |
| **DROPPED** | `CURRENT_SNAPSHOT` (old) `LEFT JOIN` `INFORMATION_SCHEMA.COLUMNS` (live) | This time we start from the **old snapshot**. If a column that used to exist has `c.COLUMN_NAME IS NULL` (no match in the live data), it means that column **used to be there but isn't anymore** → it was dropped. (Think of it like checking an old WhatsApp chat screenshot to see what message got deleted — you need the "before" picture as your starting point.) |
| **MODIFIED** | Normal `JOIN` (`INNER JOIN`) between live and snapshot | We only care about columns that exist **in both** (same table + same column name), but where `DATA_TYPE` differs between the two — meaning someone changed its type |

**Why refresh the snapshot at the end (`DELETE` + re-`INSERT`)?**
If you don't update the snapshot after logging the changes, running the procedure again would detect the **same changes all over again** (duplicate log entries). Refreshing the snapshot makes it match the current reality, so the next run only picks up **new** changes going forward.

### ✅ Key Takeaway
This is a **"diff" pattern** — commonly used far beyond Snowflake (in software version control, data reconciliation, etc.): keep a saved copy of "how things were," compare it against "how things are now," and log only the differences.

---

## 🎓 Summary Table — All 5 Real-Time Scenarios

| # | Scenario | Core Snowflake Feature Used | Key Skill Being Tested |
|---|---|---|---|
| 1 | Dynamic S3 date-folder loading | Stored Procedures, Dynamic SQL, `EXECUTE IMMEDIATE` | Building dynamic file paths from the current date |
| 2 | Valid vs. faulty record routing | `ON_ERROR = CONTINUE`, `VALIDATE()`, reverse `COPY INTO` | Error-handling and rejecting bad data gracefully |
| 3 | Schema Evolution | `INFER_SCHEMA`, `USING TEMPLATE`, `ENABLE_SCHEMA_EVOLUTION` | Handling changing file structures automatically |
| 4 | Multi-folder → multi-table pipeline | Directory Tables, Cursors, `IDENTIFIER()`, audit logging | Fully dynamic, scalable ingestion pipelines |
| 5 | Single-cell header parsing | `SPLIT()`, `ARRAY_SIZE()`, `WHILE` loops, dynamic `CREATE TABLE` | Manual parsing when built-in tools don't apply |
| 6 | DDL Change Tracking | `INFORMATION_SCHEMA.COLUMNS`, snapshot comparison, `LEFT JOIN` | Building a "diff"-based audit trail for schema changes |

**Common thread across all scenarios:** Snowflake **Stored Procedures** + **Dynamic SQL** (`EXECUTE IMMEDIATE`) are the backbone of almost every real-time automation scenario — they let you build SQL statements as text at runtime and then execute them, which is essential whenever table names, file paths, or column lists aren't known ahead of time.




# Snowflake Real-Time Scenarios — Explained Simply (With Full Code Walkthrough)

> Covers 4 real interview-style Snowflake projects, explained in plain language:
> 1. Automate Data Load & Validation in a Single Step (Stored Procedure)
> 2. Real-Time Pipeline — Snowpipe + Dynamic Tables + Alerts
> 3. Load Excel Files into Snowflake Using Snowpark (Python)
> 4. Automate Table Creation & Data Load from S3 to Snowflake Using Snowpark (Python + Task + Email Alerts)

Each scenario follows: **Business Problem → Concept → Full Code → Line-by-Line Explanation → Interview Takeaways**

---

# Scenario 1: Automate Data Load & Validation in a Single Step

## The Business Problem

Imagine a client sends you a CSV file every day. The old, painful way of handling this:

1. Load the raw file into a table (bronze/raw layer).
2. Later, someone notices bad data — missing phone numbers, duplicate customer IDs.
3. You now have to manually debug, clean, delete bad rows, and reload — wasting hours.

**The better way:** catch the problems *before* the data ever reaches your final table. Load the file into a **temporary staging table** first, run validation checks there, and only push the data into the real table if it passes. If anything fails — either the load itself or the validation — log it to an audit table so nothing silently disappears.

## The Design (Plain English)

```
Stage File (CSV) 
     │
     ▼
Temporary Table  ──► Run checks: null values? duplicates? row count zero?
     │
     ├── ✅ PASS ──► Insert into Final Table (CUSTOMER_DATA)
     │
     └── ❌ FAIL ──► Do NOT insert, just log the reason
     │
     ▼
Always write outcome (success or failure) to AUDIT_LOG table
```

This whole thing is wrapped inside **one Stored Procedure**, so it runs as a single automated unit — not five separate manual steps.

## Full Code

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

    -- Drop temp table if it exists
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
        -- Step 1: Load data into temporary table
        COPY INTO IDENTIFIER(:temp_table_name)
        FROM @SOURCE_DATA 
        FILE_FORMAT = (FORMAT_NAME='CSVONE')
        ON_ERROR = 'ABORT_STATEMENT';

        -- Step 2: Validation checks
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

    -- Step 3: Log the outcome always
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

## Line-by-Line / Block-by-Block Explanation

### 1. Procedure Declaration
```sql
CREATE OR REPLACE PROCEDURE LOAD_AND_VALIDATE()
RETURNS VARCHAR()
LANGUAGE SQL
```
This creates (or replaces) a **Snowflake Scripting stored procedure** called `LOAD_AND_VALIDATE`. It takes no input parameters, returns a `VARCHAR` (a text message summarizing what happened), and is written in plain SQL scripting (not Python/Java).

### 2. Variable Declaration
```sql
DECLARE
    temp_table_name STRING;
    total_rows_loaded INT DEFAULT 0;
    duplicate_rows INT DEFAULT 0;
    null_rows INT DEFAULT 0;
    status VARCHAR(50);
    error_message VARCHAR();
    result_message STRING;
```
Think of this as setting up "memory boxes" to hold values while the procedure runs — like local variables in any programming language. `total_rows_loaded`, `duplicate_rows`, and `null_rows` start at `0` by default, so if something skips them entirely, they're still safe numbers (not `NULL`).

### 3. Initial Setup
```sql
temp_table_name := 'TEMP_CUST_DATA';
status := 'Failed';
error_message := NULL;
```
This sets a starting/default state: assume failure until proven otherwise (`status := 'Failed'`). This is a good defensive coding habit — if something unexpected happens and none of the later logic runs, you still log "Failed" instead of an empty/blank status.

### 4. Create the Temporary Table
```sql
CREATE OR REPLACE TEMPORARY TABLE IDENTIFIER(:temp_table_name) (
   C_CUSTKEY INT, C_NAME STRING, ... C_COMMENT STRING
);
```
- `IDENTIFIER(:temp_table_name)` is a neat Snowflake trick: it lets you build a table name dynamically from a **variable** instead of hardcoding it. So if you ever wanted to reuse this procedure for a different table, you'd only change the variable, not the whole script.
- It's a **TEMPORARY** table — meaning it exists only for the current session and disappears afterward. Perfect for a "scratch pad" table used just for validation, so it doesn't clutter your database permanently.

### 5. Nested BEGIN...EXCEPTION Block (the safety net)
```sql
BEGIN
    -- Step 1: Load data
    COPY INTO ...
    -- Step 2: Validation checks
    ...
EXCEPTION
    WHEN OTHER THEN
        status := 'Failed - Execution Error';
        error_message := SQLERRM;
        result_message := error_message;
END;
```
This is exactly like a `try...catch` block in Python/Java. Everything risky (loading data, running checks) sits inside `BEGIN...END`. If **anything** throws an error — like a malformed CSV file breaking the `COPY INTO` — the `EXCEPTION WHEN OTHER` block catches it instead of crashing the whole procedure. `SQLERRM` automatically captures Snowflake's actual error message, so you don't have to guess what went wrong.

**This is the single most important design choice in this procedure** — it guarantees the procedure never dies silently. Whatever happens (data issue OR system issue), you always get a clean status and message at the end.

### 6. Step 1 — Load Data (COPY INTO)
```sql
COPY INTO IDENTIFIER(:temp_table_name)
FROM @SOURCE_DATA 
FILE_FORMAT = (FORMAT_NAME='CSVONE')
ON_ERROR = 'ABORT_STATEMENT';
```
This loads the CSV sitting in the stage `@SOURCE_DATA` into the temporary table.
- `FILE_FORMAT = (FORMAT_NAME='CSVONE')` tells Snowflake how to parse the file (delimiter, header row, etc. — pre-defined as a named file format object).
- `ON_ERROR = 'ABORT_STATEMENT'` means: if even one row is malformed, stop the entire load immediately and throw an error (which then gets caught by the `EXCEPTION` block above). This is intentional — we want zero tolerance for structurally broken files.

### 7. Step 2 — Validation Checks

**Total row count:**
```sql
total_rows_loaded := (SELECT COUNT(*) FROM IDENTIFIER(:temp_table_name));
```
Simple sanity check — did we actually get any rows at all?

**Null check:**
```sql
null_rows := (
    SELECT COUNT(*) FROM IDENTIFIER(:temp_table_name)
    WHERE C_PHONE IS NULL OR C_NATIONKEY IS NULL
);
```
Counts how many rows are missing critical fields (`C_PHONE` or `C_NATIONKEY`). You can extend this to check any column you consider mandatory.

**Duplicate check:**
```sql
duplicate_rows := (
    SELECT COUNT(*)
    FROM (
        SELECT C_CUSTKEY,
               ROW_NUMBER() OVER (PARTITION BY C_CUSTKEY ORDER BY C_CUSTKEY) AS rn
        FROM IDENTIFIER(:temp_table_name)
    ) a
    WHERE rn > 1
);
```
This is the classic **"find duplicates using ROW_NUMBER()"** pattern:
- `PARTITION BY C_CUSTKEY` groups rows by customer key.
- `ROW_NUMBER() OVER (...)` numbers each row within its group: 1, 2, 3...
- Any row numbered `rn > 1` is a *repeat* of a customer key that already appeared once — i.e., a duplicate.
- Counting those gives you the total number of duplicate rows.

### 8. Decision — Insert or Reject
```sql
IF (null_rows = 0 AND duplicate_rows = 0 AND total_rows_loaded > 0) THEN
    INSERT INTO CUSTOMER_DATA (...)
    SELECT ... FROM IDENTIFIER(:temp_table_name);
    status := 'Success';
    result_message := 'Successfully loaded ' || total_rows_loaded || ' rows with no validation errors.';
ELSE
    status := 'Failed - Validation Errors';
    error_message := 'Validation failed. ' || null_rows || ' rows with nulls, ' || duplicate_rows || ' duplicate rows found.';
    result_message := error_message;
END IF;
```
This is the actual **gate**: data only moves from the temp table into the real `CUSTOMER_DATA` table if *all three* conditions are true — no nulls, no duplicates, and at least one row was loaded. If any condition fails, nothing is inserted; instead, a descriptive error message is built explaining exactly what failed and by how many rows. This is far more useful for debugging than a generic "load failed."

### 9. Always Log to Audit Table
```sql
INSERT INTO AUDIT_LOG (
    status, load_timestamp, rows_loaded, duplicate_rows, null_rows, error_message
)
VALUES (
    :status, CURRENT_TIMESTAMP(), :total_rows_loaded, :duplicate_rows, :null_rows, :error_message
);
```
This line runs **no matter what happened above** — success, validation failure, or execution error. That's the whole point of putting it *outside* the inner `BEGIN...EXCEPTION` block: this INSERT is unconditional, so you always have a full history of every run, which is invaluable when troubleshooting "why didn't yesterday's load work?"

### 10. Return a Summary
```sql
RETURN 'Procedure execution completed with status: ' || status || '. Message: ' || result_message;
```
The procedure returns one final readable string so whoever (or whatever orchestration tool) calls it immediately knows the outcome without needing to query the audit table separately.

## What the Video Demonstrated

1. **Ran with a broken file format** → got an *execution error* (`found while expecting record delimiter`) because the file format didn't match the actual CSV structure. Logged as `Failed - Execution Error`.
2. **Fixed the file format, ran again** → this time the COPY succeeded, but validation caught 2 null rows and 1 duplicate → logged as `Failed - Validation Errors`, and **zero rows were inserted** into `CUSTOMER_DATA`.
3. **Fixed the source data and re-uploaded the file** → ran again → all validations passed → **126,000+ rows successfully inserted**, logged as `Success`.

## Interview Takeaways

| Concept | Why It Matters |
|---|---|
| `IDENTIFIER(:var)` | Lets you dynamically reference table/object names using variables — makes procedures reusable |
| `TEMPORARY` table | Safe scratch space for staging/validation that auto-cleans itself, doesn't pollute production schemas |
| Nested `BEGIN...EXCEPTION` | Mimics try/catch; ensures the procedure never crashes silently and always produces a status |
| `SQLERRM` | Built-in variable that captures the actual system error message inside an exception handler |
| `ROW_NUMBER() OVER (PARTITION BY ...)` | The standard SQL pattern for detecting duplicate records |
| Audit logging outside the try block | Guarantees you always get a log entry regardless of success/failure — critical for production pipelines |
| "Validate before you commit" pattern | A very common real-world data engineering pattern — catch issues at ingestion, not after |

---

# Scenario 2: Real-Time Pipeline — Snowpipe + Dynamic Tables + Alerts

## The Business Problem

A company has an S3 bucket where customer and item files land continuously. The requirement:
1. Ingest files automatically (Snowpipe) into a **raw/staging layer**.
2. Build a **clean layer** on top using transformations — e.g., "only keep the latest customer record" and "only keep the item with the highest price."
3. Combine the cleaned tables into one **final reporting table** with a derived metric (price per item).
4. If the final table's calculation ever fails (e.g., division by zero), **automatically email someone** instead of relying on manual monitoring.

This entire scenario is built using **Dynamic Tables**, which is Snowflake's way of automatically keeping downstream tables refreshed as source data changes — without you writing and scheduling your own transformation jobs.

## The Design (Plain English)

```
S3 Bucket (customer.csv, item.csv)
        │  (Snowpipe — not shown in detail here, assumed already ingesting)
        ▼
STG_CUSTOMER, ITEM   (raw staging tables)
        │
        ▼
CUSTOMER_DT   (Dynamic Table: dedupe → keep latest record per customer)
ITEM_DT       (Dynamic Table: dedupe → keep highest-price item per customer)
        │
        ▼
CUST_ITEM_DT  (Dynamic Table: join CUSTOMER_DT + ITEM_DT, calculate price per item)
        │
        ▼
If refresh fails → ALERT object checks refresh history every 1 min
        │
        ▼
   Sends email via Notification Integration
```

## Full Code

```sql
CREATE OR REPLACE SCHEMA PIPELINE;

-- Create Customer Table
CREATE OR REPLACE TABLE STG_Customer (
    CUST_ID STRING,
    CUST_NAME STRING,
    OUTSTANDING_AMT NUMBER,
    CRID STRING,
    LOCATION STRING,
    CUST_CREATED DATE
);

-- Insert Data
INSERT INTO STG_Customer (CUST_ID, CUST_NAME, OUTSTANDING_AMT, CRID, LOCATION, CUST_CREATED) VALUES
('C-101', 'Raman', 500, 'ABVC', 'LA', '2025-08-11'),
('C-101', 'Raman', 500, 'ABVC', 'LA', '2025-08-12'), -- in the cleansed layer we should have latest record
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
('C-101', 'a-101', 'Printer', 'Active', 4, 200), -- this row should be there 
('C-102', 'a-103', 'Ink', 'Active', 2, 300),
('C-103', 'a-103', 'Ribbon', 'Active', 3, 100),
('C-103', 'a-103', 'Ribbon', 'Active', 2, 200); -- We need this row as this is highest price 

-- STEP 2: cleansed layer
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

-- STEP 3: Final Dynamic table calculating price per item
CREATE OR REPLACE DYNAMIC TABLE CUST_ITEM_DT
  TARGET_LAG = '1 MINUTES'
  WAREHOUSE = COMPUTE_WH
AS
  SELECT c.cust_id, c.cust_name, c.crid, c.location, c.cust_created,
         a.item_id, a.item_category, a.item_status, a.price, a.counts,
         ROUND(a.price / a.counts, 2) AS Price_Per_item
  FROM CUSTOMER_DT c, ITEM_DT a
  WHERE c.cust_id = a.cust_id;

-- Simulate new incoming data (this would come from S3 via Snowpipe in real life)
INSERT INTO STG_CUSTOMER (CUST_ID, CUST_NAME, OUTSTANDING_AMT, CRID, LOCATION, CUST_CREATED)
VALUES 
('c-102', 'Megan', 3500, 'XYAZ', 'AF', '2023-08-15'),
('c-104', 'Vincet', 5000, 'ABDF', 'TX', '2023-08-15');

INSERT INTO ITEM (CUST_ID, ITEM_ID, ITEM_CATEGORY, ITEM_STATUS, COUNTS, PRICE)
VALUES 
('c-104', 'a-104', 'Oil', 'Active', 0, 500);   -- COUNTS = 0 will break the division!

-- Email notification setup
CREATE OR REPLACE NOTIFICATION INTEGRATION DT_FAILURE_ALERT
TYPE = EMAIL
ENABLED = TRUE
ALLOWED_RECIPIENTS = ('emailid@gmail.com')
COMMENT = 'Snowflake Dynamic Table Refresh Notification';

-- Alert that watches for refresh failures
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

-- Check dynamic table refresh history manually
SELECT * FROM TABLE(
    INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
        DATA_TIMESTAMP_START => DATEADD('hour', -1, CURRENT_TIMESTAMP()),
        DATA_TIMESTAMP_END   => DATEADD('hour', 0, CURRENT_TIMESTAMP()),
        NAME => 'CUST_ITEM_DT', 
        ERROR_ONLY => TRUE
    )
);

ALTER ALERT DT_FAILURE RESUME;   -- Alerts are suspended by default, just like Tasks
```

## Line-by-Line / Block-by-Block Explanation

### 1. Staging Tables
```sql
CREATE OR REPLACE TABLE STG_Customer (...);
CREATE OR REPLACE TABLE ITEM (...);
```
These represent the **raw layer** — the exact data as it would land from Snowpipe ingesting files from S3. Notice the sample data intentionally contains:
- `C-101` inserted **twice** with different `CUST_CREATED` dates (simulating an updated record arriving twice).
- Item `a-101` for `C-101` inserted **twice** at different prices (simulating a price change over time).

### 2. `CUSTOMER_DT` — Deduplication Dynamic Table
```sql
CREATE OR REPLACE DYNAMIC TABLE CUSTOMER_DT
  TARGET_LAG = DOWNSTREAM
  WAREHOUSE = COMPUTE_WH
  INITIALIZE = ON_CREATE
AS
  SELECT * FROM STG_CUSTOMER 
  QUALIFY ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY CUST_CREATED DESC) = 1;
```
A **Dynamic Table** is a special Snowflake table type that automatically re-runs its defining query and refreshes itself whenever the source data changes — you don't need a Task or a manual `MERGE` script.

- `QUALIFY ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY CUST_CREATED DESC) = 1` — this is the deduplication logic. For each `CUST_ID`, it numbers rows by creation date descending (newest = 1), then `QUALIFY` keeps only the row numbered 1 — i.e., the latest record per customer. `QUALIFY` is a Snowflake-specific clause that lets you filter directly on a window function's result (avoiding a wrapping subquery).
- `TARGET_LAG = DOWNSTREAM` — this means "don't refresh on your own fixed schedule; instead, refresh whenever a *downstream* dynamic table (one that depends on you) needs fresh data." It's a way of saying "I don't drive my own refresh timing — whoever consumes me does."
- `WAREHOUSE = COMPUTE_WH` — which virtual warehouse performs the refresh computation.
- `INITIALIZE = ON_CREATE` — as soon as the table is created, immediately populate it with the full result set (as opposed to `ON_SCHEDULE`, which would wait for the first scheduled refresh before populating).

### 3. `ITEM_DT` — Keep Only the Highest-Priced Item
```sql
CREATE OR REPLACE DYNAMIC TABLE ITEM_DT
  TARGET_LAG = DOWNSTREAM
  WAREHOUSE = COMPUTE_WH
AS
  SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY CUST_ID ORDER BY PRICE DESC) AS rn
    FROM ITEM 
  ) t
  WHERE rn = 1;
```
Same deduplication idea, but this time using a classic subquery + `WHERE rn = 1` instead of `QUALIFY` (both approaches are equivalent — the video shows both styles). For each customer, only the item row with the **highest price** survives.

### 4. `CUST_ITEM_DT` — The Final Combined Table
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
This joins the two cleaned dynamic tables (`CUSTOMER_DT` and `ITEM_DT`) — the comma between tables plus a `WHERE` clause is old-style SQL for an inner join. It then calculates a derived metric: `price / counts`, rounded to 2 decimals.

- `TARGET_LAG = '1 MINUTES'` — this table sits at the **top of the chain**, so it needs an actual, real refresh schedule rather than `DOWNSTREAM` (since nothing is downstream of it). Every 1 minute, Snowflake checks if source data changed and refreshes accordingly. Because `CUSTOMER_DT` and `ITEM_DT` were set to `DOWNSTREAM`, this 1-minute schedule on the *final* table effectively drives refreshes for the entire chain.

**Key concept: Dynamic Table Dependency Graph.** Snowflake automatically tracks that `CUST_ITEM_DT` depends on `CUSTOMER_DT` and `ITEM_DT`. Whichever table has the "real" schedule (here, the 1-minute one) pulls the `DOWNSTREAM` tables along with it during refresh — you don't need to manually orchestrate the order.

### 5. The Deliberate Failure — Division by Zero
```sql
INSERT INTO ITEM (CUST_ID, ITEM_ID, ITEM_CATEGORY, ITEM_STATUS, COUNTS, PRICE)
VALUES ('c-104', 'a-104', 'Oil', 'Active', 0, 500);
```
This inserts a row where `COUNTS = 0`. Since `CUST_ITEM_DT`'s formula is `price / counts`, this creates a **division-by-zero error** the next time the dynamic table refreshes — demonstrating a realistic scenario where bad data silently breaks a pipeline unless you're actively monitoring for it.

### 6. Notification Integration
```sql
CREATE OR REPLACE NOTIFICATION INTEGRATION DT_FAILURE_ALERT
TYPE = EMAIL
ENABLED = TRUE
ALLOWED_RECIPIENTS = ('emailid@gmail.com')
COMMENT = 'Snowflake Dynamic Table Refresh Notification';
```
This is a Snowflake object that authorizes sending emails to a specific, whitelisted set of recipients. You must create this before you can call `SYSTEM$SEND_EMAIL`.

### 7. The Alert Object
```sql
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
))
THEN CALL SYSTEM$SEND_EMAIL(...);
```
An **Alert** is a Snowflake object made of three parts: a schedule (how often to check), a condition (`IF EXISTS ...` — did anything match?), and an action (`THEN CALL ...` — what to do if the condition is true).

- `INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(...)` is a **table function** that returns the refresh history of a dynamic table — including any refreshes that failed.
- `ERROR_ONLY => TRUE` filters this history down to only the failed refresh attempts.
- `DATA_TIMESTAMP_START` / `DATA_TIMESTAMP_END` define the lookback window (here, the last 1 hour).
- If this query returns **any rows at all** (`EXISTS`), it means there was at least one failed refresh recently — trigger the `THEN` action.
- `SYSTEM$SEND_EMAIL('DT_FAILURE_ALERT', 'emailid', 'subject', 'body')` is a built-in Snowflake function that sends an email using the notification integration you created, to the given recipient, with a subject and body.

### 8. Resuming the Alert
```sql
ALTER ALERT DT_FAILURE RESUME;
```
Just like **Tasks**, newly created **Alerts** are **suspended by default**. You must explicitly resume them, or they will never actually run — a very common "gotcha" that catches people off guard (the presenter even calls this out directly: "don't forget to resume it").

## What the Video Demonstrated

1. Created the staging tables and the chain of 3 dynamic tables.
2. Confirmed `CUSTOMER_DT` correctly kept only the latest record per customer, and `ITEM_DT` kept only the highest-priced item.
3. Inserted a bad row with `COUNTS = 0`.
4. Watched `CUST_ITEM_DT`'s refresh fail with error code `1051` (division by zero) after its 1-minute schedule triggered.
5. Set up the notification integration + alert, resumed the alert, and received an actual email within about a minute of the failure.
6. Fixed the bad row (`UPDATE ITEM SET COUNTS = 5 WHERE ...`) to stop future failures.

## Interview Takeaways

| Concept | Why It Matters |
|---|---|
| Dynamic Tables | Declarative way to build multi-stage transformation pipelines without writing/scheduling your own MERGE logic |
| `TARGET_LAG = DOWNSTREAM` | Lets intermediate tables inherit their refresh timing from whichever table consumes them, avoiding redundant schedules |
| `QUALIFY` | Snowflake-specific clause to filter directly on window function results |
| Dynamic Table dependency graph | Snowflake automatically manages refresh order across dependent dynamic tables |
| `INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY()` | Table function to audit dynamic table refresh success/failure history |
| Alerts vs. Tasks | Alerts are built for "check condition → notify," commonly paired with `SYSTEM$SEND_EMAIL`; both are suspended by default and must be resumed |
| Notification Integration | Required setup object before you can send emails from Snowflake |

---

# Scenario 3: Load Excel Files into Snowflake Using Snowpark (Python)

## The Business Problem

Snowflake's native `COPY INTO` command works great for CSV/JSON/Parquet — but it **cannot directly load Excel (.xlsx) files**. If a business team keeps sending Excel files instead of CSVs, you need another way in. The answer: use **Snowpark Python** together with the `pandas` and `openpyxl` libraries to read the Excel file and push it into a Snowflake table — all inside a stored procedure so it can be called and scheduled like any other Snowflake object.

## The Design (Plain English)

```
Excel file sitting in an internal stage (@EXCEL_DATA)
        │
        ▼
Snowpark Python Procedure:
   1. Download file from stage to local temp storage
   2. Read it into a pandas DataFrame (pandas understands .xlsx)
   3. Convert pandas DataFrame → Snowpark DataFrame
   4. Write Snowpark DataFrame into a real Snowflake table
   5. Log the outcome (success/failure + row count) to a log table
```

## Full Code

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
        # Step 2: Get file from stage
        file_path = f"@EXCEL_DATA/{file_name}"  
        session.file.get(file_path, '/tmp')  # download file from stage to local temp folder
        local_path = f"/tmp/{file_name}" 

        # Step 3: Read Excel into pandas dataframe
        panda_df = pd.read_excel(local_path)

        # Step 4: Convert pandas dataframe to Snowpark dataframe
        snow_df = session.create_dataframe(panda_df)

        try:
            # Step 5: Write dataframe to Snowflake table
            snow_df.write.save_as_table(target_table, mode="overwrite")

            # Step 6: Log success with row count
            row_count = len(panda_df)
            log_status(session, file_name, target_table, "SUCCESS", None, row_count)
            return f"{row_count} rows into table '{target_table}' Loaded Successfully"

        except Exception as e:
            # Step 7: Log failure if table write fails
            log_status(session, file_name, target_table, "FAILED", str(e), 0)
            return f"Failed to write table: {e}"

    except Exception as e:
        # Step 8: Log top-level failure
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

## Line-by-Line / Block-by-Block Explanation

### 1. Procedure Header
```sql
CREATE OR REPLACE PROCEDURE LOAD_EXCEL_FILES(file_name STRING, target_table STRING) 
RETURNS STRING
LANGUAGE PYTHON
RUNTIME_VERSION = '3.9'
PACKAGES = ('snowflake-snowpark-python', 'pandas', 'openpyxl')
HANDLER = 'main'
```
- This is a **Python stored procedure** — unlike Scenario 1, it's written in Python instead of SQL, which is necessary because Excel parsing needs the `pandas` + `openpyxl` libraries.
- It takes two parameters: `file_name` (which Excel file to load) and `target_table` (where to load it) — this makes it reusable for any file/table combination, not hardcoded to one.
- `PACKAGES = (...)` tells Snowflake which pre-approved Anaconda-hosted Python packages to make available inside the sandboxed execution environment.
- `HANDLER = 'main'` tells Snowflake which Python function to actually call when the procedure runs — Snowflake will look for a function named `main` in the code below.

### 2. Function Signature
```python
def main(session: Session, file_name: str, target_table: str) -> str:
```
Every Snowpark Python procedure's handler function automatically receives a `session` object as its first argument — this is your live connection to Snowflake, used to run SQL, read/write tables, and access stage files, all without needing separate connection credentials.

### 3. Download the File From Stage
```python
file_path = f"@EXCEL_DATA/{file_name}"  
session.file.get(file_path, '/tmp')
local_path = f"/tmp/{file_name}"
```
Snowpark's Python sandbox runs in an isolated environment that has its own local filesystem (`/tmp`). Since `pandas.read_excel()` needs an actual local file (it can't read directly from a Snowflake stage), you must first **download** the file from the stage into this local temp folder using `session.file.get(...)`.

### 4. Read Excel Into pandas
```python
panda_df = pd.read_excel(local_path)
```
This is standard `pandas` — it reads the Excel file into a DataFrame, automatically inferring the column headers and data types. This is the whole reason Snowpark/Python is needed here: `pandas.read_excel()` has no native SQL equivalent in Snowflake.

### 5. Convert pandas → Snowpark DataFrame
```python
snow_df = session.create_dataframe(panda_df)
```
A `pandas` DataFrame lives entirely in local Python memory. To actually get this data *into* Snowflake, you convert it into a **Snowpark DataFrame**, which is a lazy, distributed representation Snowflake understands and can push down into SQL operations.

### 6. Write to a Real Snowflake Table
```python
snow_df.write.save_as_table(target_table, mode="overwrite")
```
This physically writes the Snowpark DataFrame into a Snowflake table named by the `target_table` parameter. `mode="overwrite"` means: if the table already exists, replace its contents entirely (other options include `"append"` to add rows without deleting existing ones).

### 7. Nested Try/Except for Precise Error Isolation
```python
try:
    # ... download + read + convert ...
    try:
        # write to table
        ...
    except Exception as e:
        # only the WRITE failed
        log_status(session, file_name, target_table, "FAILED", str(e), 0)
        return f"Failed to write table: {e}"
except Exception as e:
    # something earlier failed (download or read)
    log_status(session, file_name, target_table, "FAILED", traceback.format_exc()[:1500], 0)
    return f"Procedure could not execute. Error: {str(e)}"
```
This mirrors the nested-`BEGIN...EXCEPTION` pattern from Scenario 1, but in Python: an **outer** try/except catches anything going wrong in the download/read/convert steps, while an **inner** try/except specifically isolates failures during the *table write* step. This precision is useful: it lets you distinguish "the Excel file itself is broken" from "the table write failed" (e.g., due to a schema mismatch or permissions issue) — two very different problems requiring different fixes.
- `traceback.format_exc()[:1500]` captures the **full Python stack trace** (not just the error message) so you have maximum debugging detail, but truncated to 1500 characters so it doesn't overflow the log table's column size.

### 8. The Logging Helper Function
```python
def log_status(session, file_name, target_table, status, error_message=None, row_count=0):
    try:
        insert_stmt = f"""
            INSERT INTO EXCEL_LOAD_LOGS (...)
            VALUES (
                '{file_name}', '{target_table}', '{status}',
                {'NULL' if error_message is None else "'" + error_message.replace("'", "''") + "'"},
                {row_count}, CURRENT_TIMESTAMP()
            )
        """
        session.sql(insert_stmt).collect()
    except Exception as log_err:
        print("Log insert failed:", log_err)
```
This is a reusable helper (called from three different places above) that writes to the `EXCEL_LOAD_LOGS` audit table.

- `{'NULL' if error_message is None else "'" + error_message.replace("'", "''") + "'"}` — this is a common but important SQL-injection-safety pattern: if there's no error, insert the literal SQL keyword `NULL`; otherwise, wrap the error message in quotes, but first **escape any single quotes** inside it (`replace("'", "''")`) so an error message containing an apostrophe doesn't break the SQL statement or allow injection.
- `session.sql(insert_stmt).collect()` — builds the SQL string and executes it. `.collect()` actually triggers execution (Snowpark queries are otherwise lazy).
- The **whole logging call is itself wrapped in try/except** — a subtle but smart detail: if the *logging* step itself fails for some reason, it shouldn't crash the entire procedure; it just prints a message instead.

## Interview Takeaways

| Concept | Why It Matters |
|---|---|
| Python Stored Procedures | Needed whenever native SQL/COPY INTO can't handle a file format (like Excel) |
| `PACKAGES` clause | Declares which sandboxed Python libraries the procedure can use |
| `session.file.get()` | Downloads a stage file into the local Python sandbox filesystem |
| pandas → Snowpark DataFrame conversion | The bridge between "regular Python data" and "Snowflake-native data" |
| `save_as_table(mode="overwrite")` | Writes a DataFrame into a real table; overwrite vs. append matters |
| Nested try/except | Isolates *which stage* of the pipeline failed for more precise debugging |
| Escaping quotes before dynamic SQL insert | Prevents broken SQL / injection issues when logging arbitrary error text |

---

# Scenario 4: Automate Table Creation & Data Load from S3 to Snowflake Using Snowpark

## The Business Problem

Multiple teams (finance, marketing, sales, HR) drop CSV files into a shared S3 bucket, each team in its own folder. Requirements:
1. Automatically detect new folders/files.
2. If a folder is new, automatically **create a matching table** (without you manually writing `CREATE TABLE` for every possible team).
3. Load only **new** files — never reload a file that was already processed.
4. Run this on a **schedule** (via a Task) so it's fully hands-off.
5. If anything breaks, get an **email notification** automatically.

This is essentially a self-service, self-expanding ingestion framework — a very common "warehouse automation" interview scenario.

## The Design (Plain English)

```
S3 Bucket
 ├── customer/customer.csv
 ├── nation/nation.csv
 ├── region/region.csv
 └── supplier/supplier.csv
        │
        ▼
Snowpark Python Procedure (runs on a schedule via TASK):
   1. LIST all files in the external stage
   2. Compare against LOG_TABLE — skip files already processed
   3. For new files: extract folder name (→ becomes table name) and file name
   4. If table doesn't exist yet → CREATE TABLE automatically using INFER_SCHEMA
   5. COPY INTO the table
   6. INSERT a row into LOG_TABLE marking this file as processed
        │
        ▼
   If the procedure/task fails → SNS/email notification fires automatically
```

## Full Code

```python
import snowflake.snowpark as snowpark
from snowflake.snowpark.functions import col

def main(session: snowpark.Session) -> str:
    stage = "@S3_STAGE"
    file_format = "csv_format"
    created_tables = []
    log_messages = []  # To store all messages for final output

    # List all files from the stage
    stage_files_df = session.sql(f"LIST {stage}").collect()

    # check already loaded files from Audit table
    loaded_files_df = session.table("LOG_TABLE").select("FILE_NAME").collect()
    already_loaded_files = {row["FILE_NAME"] for row in loaded_files_df}

    for row in stage_files_df:
        file_path = row['name']  # e.g., s3://bucket/folder/file.csv

        # if files are already loaded then we skip those files here
        if file_path in already_loaded_files:
            msg = f"Skipping already loaded file: {file_path}"
            print(msg)
            log_messages.append(msg)
            continue

        # get the complete file path and split based on /
        parts = file_path.split('/')

        # extracting folder, file name and setting folder = table name
        if len(parts) > 1:
            folder = parts[-2]
            file_name = parts[-1]
            table_name = folder.upper()

            # setting up complete path for the copy into load
            full_path = f"{stage}/{folder}/{file_name}"

            # Check if table already exists
            exists_result = session.sql(f"""
                SELECT COUNT(*) AS count 
                FROM INFORMATION_SCHEMA.TABLES 
                WHERE TABLE_NAME = '{table_name}' 
                AND TABLE_SCHEMA = CURRENT_SCHEMA()
            """).collect()
            
            exists = int(exists_result[0]['COUNT']) if exists_result else 0

            if exists == 0:
                # Create table automatically using INFER_SCHEMA
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

            # load data into table using COPY INTO
            copy_sql = f"""
                COPY INTO {table_name}
                FROM {full_path}
                FILE_FORMAT = (FORMAT_NAME = '{file_format}')
                MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
                ON_ERROR = ABORT_STATEMENT
            """
            session.sql(copy_sql).collect()
            log_messages.append(f"Loaded data into table: {table_name} from {file_name}")

            # log the processed file so we skip it next time
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

**Supporting setup shown in the video (SQL side):**
```sql
CREATE OR REPLACE SCHEMA AUTO;

CREATE OR REPLACE TABLE LOG_TABLE (
    FOLDER_NAME STRING,
    FILE_NAME STRING,
    LOAD_TIMESTAMP TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

CREATE OR REPLACE FILE FORMAT csv_format
  TYPE = 'CSV'
  PARSE_HEADER = TRUE;   -- required for INFER_SCHEMA (cannot use SKIP_HEADER)

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

## Line-by-Line / Block-by-Block Explanation

### 1. Setup — Listing Stage Files
```python
stage_files_df = session.sql(f"LIST {stage}").collect()
```
`LIST @S3_STAGE` is a Snowflake command that returns every file currently sitting in the stage, including its full path (e.g., `s3://bucket/customer/customer.csv`), size, and last-modified time. This is how the procedure "discovers" what exists in S3 without you telling it in advance.

### 2. Building the "Already Loaded" Lookup Set
```python
loaded_files_df = session.table("LOG_TABLE").select("FILE_NAME").collect()
already_loaded_files = {row["FILE_NAME"] for row in loaded_files_df}
```
- `session.table("LOG_TABLE")` reads the audit table as a Snowpark DataFrame.
- `{row["FILE_NAME"] for row in loaded_files_df}` is a Python **set comprehension** — it builds a Python `set` of every filename already logged. Sets give **O(1) fast lookups**, which matters if you have thousands of files to check against.

### 3. The Main Loop — Skip Already-Processed Files
```python
for row in stage_files_df:
    file_path = row['name']
    if file_path in already_loaded_files:
        msg = f"Skipping already loaded file: {file_path}"
        log_messages.append(msg)
        continue
```
For every file found in the stage, check if it's already in the "processed" set. If yes, `continue` immediately jumps to the next file — this is the **idempotency** mechanism that prevents duplicate loads if the procedure runs repeatedly (e.g., every minute via a Task) and finds the same old files still sitting there.

### 4. Extracting Folder Name → Table Name
```python
parts = file_path.split('/')
if len(parts) > 1:
    folder = parts[-2]
    file_name = parts[-1]
    table_name = folder.upper()
```
Given a path like `s3://bucket/customer/customer.csv`, splitting on `/` gives a list of parts. `parts[-1]` is the last element (`customer.csv` — the file name), and `parts[-2]` is the second-to-last (`customer` — the folder name). The folder name becomes the table name (uppercased, since Snowflake unquoted identifiers are case-insensitive and stored as uppercase by convention).

**This is the clever trick of the whole scenario:** the S3 folder structure itself dictates the Snowflake table structure — no manual mapping needed. Drop a new folder called `sales/`, and the next run will create a `SALES` table automatically.

### 5. Checking If the Table Already Exists
```python
exists_result = session.sql(f"""
    SELECT COUNT(*) AS count 
    FROM INFORMATION_SCHEMA.TABLES 
    WHERE TABLE_NAME = '{table_name}' 
    AND TABLE_SCHEMA = CURRENT_SCHEMA()
""").collect()

exists = int(exists_result[0]['COUNT']) if exists_result else 0
```
`INFORMATION_SCHEMA.TABLES` is Snowflake's built-in metadata catalog — querying it for a matching `TABLE_NAME` tells you whether that table already exists in the current schema. If the count is `0`, the table doesn't exist yet and needs to be created.

### 6. Auto-Creating the Table with `INFER_SCHEMA`
```python
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
```
This is the standout Snowflake feature in this scenario: `INFER_SCHEMA` is a table function that **reads the actual file(s) in a stage location** and detects the column names and data types automatically — no need to know in advance what columns any given team's CSV will have.
- `INFER_SCHEMA(LOCATION => '...', FILE_FORMAT => '...')` scans the file and returns one row per detected column (name, type, nullable, etc.).
- `OBJECT_CONSTRUCT(*)` turns each detected column's metadata into a JSON object.
- `ARRAY_AGG(...)` collects all those column-definition objects into a single array.
- `CREATE TABLE ... USING TEMPLATE (...)` is special CREATE TABLE syntax that accepts this array of inferred column definitions and builds the table structure from it directly — effectively "create a table that matches whatever columns this file has."

This is why the file format was created with `PARSE_HEADER = TRUE` instead of `SKIP_HEADER = 1` — `INFER_SCHEMA` specifically needs `PARSE_HEADER` to correctly read column names from the CSV header row.

### 7. Loading the Data
```python
copy_sql = f"""
    COPY INTO {table_name}
    FROM {full_path}
    FILE_FORMAT = (FORMAT_NAME = '{file_format}')
    MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE
    ON_ERROR = ABORT_STATEMENT
"""
session.sql(copy_sql).collect()
```
Standard `COPY INTO`, but note `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` — this tells Snowflake to match CSV columns to table columns **by name** (ignoring case) rather than strictly by position. This pairs naturally with the auto-inferred schema, since column order in the file and table should already line up, but this adds a layer of safety.

### 8. Logging the Processed File
```python
session.sql(f"""
    INSERT INTO LOG_TABLE (FOLDER_NAME, FILE_NAME)
    VALUES ('{folder}', '{file_path}')
""").collect()
```
After a successful load, the file's full path is recorded in `LOG_TABLE`. This is exactly what feeds the `already_loaded_files` set the *next* time the procedure runs — completing the idempotency loop.

### 9. Returning a Result as a DataFrame
```python
message = "\n".join(log_messages)
return session.create_dataframe([[message]], schema=["RESULT"])
```
Unlike Scenario 3 (which returned a plain string), this handler returns a **Snowpark DataFrame** — a valid alternative return type for a Python procedure, useful when you want the output to look like a query result set (one row, one `RESULT` column) rather than a raw string.

### 10. The Orchestration Layer — Task + Error Notification
```sql
CREATE OR REPLACE TASK autoload_tables
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = '1 MINUTE'
  ERROR_INTEGRATION = task_error_notification
AS
  CALL SP_AUTO_S3_TABLES();

ALTER TASK autoload_tables RESUME;
```
This is what makes the whole thing "automated" rather than something you run manually:
- `SCHEDULE = '1 MINUTE'` — Snowflake automatically calls the procedure every minute.
- `ERROR_INTEGRATION = task_error_notification` — if the task's procedure call throws an unhandled error, Snowflake automatically publishes a failure message to the configured **AWS SNS topic**, which (once subscribed) emails the team.
- `ALTER TASK ... RESUME` — same rule as before: **Tasks are suspended by default** and must be explicitly resumed, or the schedule will simply never fire.

### 11. AWS-Side Setup (Storage Integration + SNS)
Two AWS integrations were configured in the video, both following the same underlying pattern — a **secure handshake** between AWS and Snowflake:
1. **Storage Integration**: lets Snowflake read S3 files without needing to hardcode AWS access keys/secret keys — instead, Snowflake assumes an IAM role that trusts a specific external ID, and that IAM role has S3 read permissions.
2. **Notification Integration (type QUEUE, provider AWS_SNS)**: lets a Snowflake Task automatically publish failure messages to an SNS topic, which is subscribed to an email address — so any task failure results in an automatic email, without needing an Alert object like Scenario 2 used.

Both integrations require: create/identify an IAM role in AWS → get its ARN and an external ID from Snowflake's `DESCRIBE INTEGRATION` output → paste those into the AWS IAM role's **trust policy** — completing the two-way trust relationship.

## What the Video Demonstrated

1. Ran the task for the first time — it processed all 5 existing folders/files, auto-creating 5 matching tables and logging each into `LOG_TABLE`.
2. Added a brand-new folder (`line_item/`) with a new file — the *very next scheduled run* automatically detected it, created a `LINE_ITEM` table, and loaded the data — with zero code changes required.
3. Deliberately broke the stage reference (typo'd the stage name) to simulate a failure — the task failed, and an SNS-driven email notification arrived automatically with the actual error message.

## Interview Takeaways

| Concept | Why It Matters |
|---|---|
| `LIST @stage` | Programmatically discover what files exist in a stage |
| Set-based lookup for "already processed" | Efficient idempotency check to avoid reloading the same file twice |
| Folder name → table name convention | A powerful pattern for fully automated, self-expanding ingestion frameworks |
| `INFORMATION_SCHEMA.TABLES` existence check | Standard way to conditionally create objects only if they don't already exist |
| `INFER_SCHEMA` + `CREATE TABLE ... USING TEMPLATE` | Lets Snowflake auto-detect and build a table schema directly from a file, with zero manual DDL |
| `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` | Loads data by column name instead of position — safer when schemas are inferred dynamically |
| Snowpark Python for orchestration logic | Python is better suited than pure SQL for looping, string parsing, and conditional branching logic like this |
| Task `ERROR_INTEGRATION` vs. Alert-based monitoring | Two different but equally valid ways to get automatic failure notifications — Tasks have a built-in error-integration hook; Dynamic Tables/other objects can be monitored via a separate Alert object (see Scenario 2) |
| Storage Integration & Notification Integration (SNS) | The standard two-way trust pattern for connecting Snowflake securely to AWS resources, without hardcoding credentials |
| Tasks are suspended by default | A near-universal Snowflake "gotcha" — always remember `ALTER TASK ... RESUME` (same applies to Alerts and Pipes have the opposite default — Snowpipes start running automatically) |

---

# Cross-Scenario Summary — Patterns That Repeat

These four scenarios, while different in purpose, share several recurring Snowflake design patterns worth remembering for interviews:

1. **Validate-before-commit staging pattern** (Scenario 1): load into a temp/staging object first, check data quality, only then move into the "real" table.
2. **Always log the outcome, success or failure** (all 4 scenarios): audit tables are a non-negotiable part of production pipelines — you should never have to guess why yesterday's load didn't work.
3. **Nested error handling** (Scenarios 1 & 3): isolate *which specific step* failed (load vs. validation vs. write) so error messages are actionable, not generic.
4. **Suspended-by-default objects**: Tasks and Alerts both start suspended and must be explicitly resumed — Snowpipes, by contrast, start running immediately. Always double-check object state after creation.
5. **Automated failure notification** via either:
   - **Alerts** + `SYSTEM$SEND_EMAIL` + a Notification Integration (`TYPE = EMAIL`) — good for monitoring conditions on data/objects like Dynamic Tables.
   - **Task `ERROR_INTEGRATION`** + AWS SNS + a Notification Integration (`TYPE = QUEUE`) — good for monitoring scheduled task execution failures.
6. **Dynamic/parameterized object names** using `IDENTIFIER(:variable)` (SQL) or f-strings (Python) — makes procedures reusable instead of hardcoded to one table.
7. **Schema-on-read automation** via `INFER_SCHEMA` — removes the need to manually define table DDL when source files are unpredictable or numerous.
8. **Snowpark Python bridges the gap** wherever pure SQL can't do the job — reading non-native file formats (Excel), complex looping/branching logic, or dynamic multi-table orchestration.
