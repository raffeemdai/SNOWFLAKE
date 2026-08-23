
snowflake iceberg tables

https://medium.com/snowflake/snowflake-managed-vs-self-managed-iceberg-tables-what-actually-determines-the-difference-c5615ba0d280


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
