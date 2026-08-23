# Snowflake Real-Time Interview Questions & Answers
### (Genuine scenario-based questions compiled from LinkedIn posts, technical blogs, and community discussions)

This is **not** a generic dump of textbook Snowflake questions. These are the kind of **"what would you actually do"** situational questions that real interviewers ask to check if you've genuinely worked on a live Snowflake project — not just memorized definitions.

Each question includes: the situation, a simple answer, and (where useful) the SQL/command involved.

---

## Section 1: Data Loading & File Handling Scenarios

### Q1. You already loaded a file into a table. The client sends the exact same file again (or you need to reload it for testing). What happens, and how do you force it to load again?

**What happens by default:** Snowflake keeps **load metadata** for every file tied to a table. When you run `COPY INTO` again with the same file, Snowflake checks this metadata, recognizes the file was already loaded (same checksum, unchanged), and **silently skips it** — no error, no duplicate rows, but also no reload.

**How to force a reload:**
```sql
COPY INTO my_table
FROM @my_stage
FILE_FORMAT = (FORMAT_NAME = 'CSVTYPE')
FORCE = TRUE;
```
`FORCE = TRUE` tells Snowflake to ignore the load history and reload the file anyway — useful for performance testing or when you deliberately want to re-ingest data. Be careful: this **will duplicate rows** if the table wasn't truncated first.

**Good follow-up point to mention in an interview:** This load metadata is only reliably tracked for about **64 days**. After that window, Snowflake may not be 100% sure whether a file was already loaded, especially if the table was created more than 64 days ago. Truncating or recreating the table also resets this metadata.

---

### Q2. Files are arriving in an S3 bucket continuously throughout the day. Would you use Snowpipe or a scheduled `COPY INTO` (batch load)? How do you decide?

**Simple answer:** It depends on **how urgently** the data is needed and **how many files** are arriving.

| Use Snowpipe when... | Use scheduled COPY INTO (batch/Task) when... |
|---|---|
| Files need to be available in near real-time (minutes) | A few-hours delay is acceptable |
| Files arrive continuously / unpredictably | Files arrive in predictable batches (e.g., once a day at midnight) |
| You want event-driven, serverless loading (via S3 event notifications / SQS) | You want to control compute cost precisely using your own warehouse |

**Cost angle interviewers look for:** Snowpipe charges a **per-file processing fee** on top of the actual compute used. If you have a very high volume of very small files, this per-file fee can add up and make Snowpipe more expensive than a scheduled `COPY INTO` running on a small warehouse. So for **high file counts with looser SLAs**, batch loading is often cheaper.

---

### Q3. Files are dropped into S3 in a folder structure organized by date (`year/month/day/`) and the folder path changes every day. How do you automate loading without hardcoding the path daily?

This is a very common real scenario question. The expected answer:
1. Extract year, month, and day from `CURRENT_DATE` inside a **stored procedure**.
2. Concatenate these parts to dynamically build the folder path (e.g., `2025/06/20/`).
3. Build the `COPY INTO` statement as a **string** (Dynamic SQL) using this path.
4. Run it with `EXECUTE IMMEDIATE`.
5. Wrap the whole thing in a stored procedure and schedule it with a **Task** so it runs automatically every day without manual intervention.

```sql
CREATE OR REPLACE PROCEDURE DYNAMIC_LOAD()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
    yr STRING DEFAULT TO_CHAR(CURRENT_DATE, 'YYYY');
    mn STRING DEFAULT TO_CHAR(CURRENT_DATE, 'MM');
    dy STRING DEFAULT TO_CHAR(CURRENT_DATE, 'DD');
    copy_sql STRING;
BEGIN
    copy_sql := 'COPY INTO CUSTOMER FROM @DYNAMIC_STAGE/' || yr || '/' || mn || '/' || dy || '/';
    EXECUTE IMMEDIATE :copy_sql;
    RETURN copy_sql;
END;
$$;
```

---

### Q4. A client's file has bad/malformed rows mixed in with good rows. The requirement: load the good rows, and send the bad rows back to the client for correction. How do you design this?

1. Load with `ON_ERROR = CONTINUE` so bad rows are skipped instead of failing the whole load:
   ```sql
   COPY INTO CUSTOMER FROM @AWS_STAGE ON_ERROR = CONTINUE;
   ```
2. Use the built-in `VALIDATE()` table function to see exactly which rows failed and why, referencing the last load job:
   ```sql
   SELECT * FROM TABLE(VALIDATE(CUSTOMER, JOB_ID => '_last'));
   ```
3. Save those failed rows into a separate `FAULTY_RECORDS` table.
4. Export that table back out to S3 as a CSV using `COPY INTO` in the reverse direction (table → stage), typically with `SINGLE = TRUE` so it produces one clean file instead of Snowflake's default multi-file split.
5. Wrap steps 1–4 in a reusable stored procedure (parameterized by table name and stage names) so the same logic works for any table, and optionally trigger it on a schedule or via a Task.

**Interviewer follow-up to expect:** *"What's the difference between `ON_ERROR = CONTINUE`, `SKIP_FILE`, and `ABORT_STATEMENT`?"*
- `CONTINUE` → skip only the bad rows, load everything else.
- `SKIP_FILE` → if a file has any error, skip the **entire file**.
- `ABORT_STATEMENT` (default) → stop the whole load the moment any error is hit.

---

### Q5. The client's CSV file structure keeps changing — sometimes a new column is added, sometimes columns are reordered. Your pipeline shouldn't break every time this happens. How do you handle it?

This is the **Schema Evolution / Schema Detection** scenario:
1. Use `INFER_SCHEMA()` to auto-detect a file's column names and data types instead of hardcoding them.
2. Auto-create the table using `CREATE TABLE ... USING TEMPLATE` based on the inferred schema.
3. Load with `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE` so columns are matched **by name**, not by position — this alone solves the "reordered columns" problem.
4. For genuinely **new** columns appearing in later files, turn on:
   ```sql
   ALTER TABLE CUSTOMER_DATA SET ENABLE_SCHEMA_EVOLUTION = TRUE;
   ALTER FILE FORMAT CSVTYPE SET ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE;
   ```
   Now, when a file with extra columns is loaded, Snowflake **automatically adds the new column** to the table instead of throwing an error. Old rows simply get `NULL` for that new column.

**Good thing to mention:** This feature only works with `COPY INTO` / Snowpipe — it does **not** apply to plain `INSERT` statements.

---

### Q6. An S3 bucket has multiple folders (customer, orders, region...), each meant to load into its own matching Snowflake table — and tomorrow a brand-new folder might get added. How do you build this so it needs zero code changes when a new folder shows up?

1. Enable a **Directory Table** on the stage (`DIRECTORY = (ENABLE = true, AUTO_REFRESH = TRUE)`) — this gives you a queryable list of every file's relative path.
2. Loop through all paths using a **CURSOR** inside a stored procedure.
3. From each path, split out the **folder name** (→ becomes the table name) and the **file name**.
4. Check an **audit/log table** first — if this exact file path was already processed, skip it (`CONTINUE`).
5. Check `INFORMATION_SCHEMA.TABLES` to see if a table for that folder already exists.
   - If not, auto-create it using `INFER_SCHEMA` + `CREATE TABLE ... USING TEMPLATE`, referencing the table name dynamically via `IDENTIFIER(?)`.
6. Load the file with `COPY INTO IDENTIFIER(?) FROM ? MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE ON_ERROR = 'SKIP_FILE'`.
7. Insert an entry into the audit table marking the file as processed.

**Why this matters to interviewers:** it tests whether you understand **Directory Tables, Cursors, dynamic SQL with `IDENTIFIER()`**, and idempotent pipeline design (not reprocessing the same file twice) — all skills that come up constantly in production data engineering.

---

### Q7. A vendor sends a CSV where all the column names are jammed into a single cell in the header row (e.g. `col1|col2|col3`), instead of proper separate header cells. `INFER_SCHEMA` doesn't work on this. What do you do?

Since Snowflake's automatic schema detection can't parse this format, you fall back to **manual string parsing**:
1. Read the raw header line as one text value using `$1` (meaning "give me everything in the first column/cell as raw text").
2. `SPLIT()` that text by the delimiter (e.g., `|`) into an array of column names.
3. Use `ARRAY_SIZE()` to count how many columns exist.
4. Loop through the array (`WHILE` loop) to build a `"column_name" STRING, ...` definition string.
5. Dynamically build and run a `CREATE TABLE` statement using that string.
6. Load the actual data separately with `COPY INTO ... SKIP_HEADER = 1` (skipping the header row this time, since you already extracted the column names).

This tests whether a candidate can fall back to **manual parsing + dynamic SQL** when the built-in tools don't fit — a realistic "vendor gave us a weird file" situation.

---

## Section 2: Performance, Cost & Troubleshooting Scenarios

### Q8. Query performance has degraded noticeably as data volume grew. Walk me through how you'd diagnose and fix it.

Expected structured answer:
1. Open the **Query Profile** in Snowsight for the slow query and look for expensive operations — full table scans, large joins, or "spilling" to local/remote disk (a sign the warehouse is too small for the data volume).
2. Check `QUERY_HISTORY` to see if this is a one-off slow query or a consistent pattern.
3. Look at **partition pruning** — if a query is scanning far more micro-partitions than necessary, a **clustering key** on the frequently filtered/joined column may help.
4. Consider **resizing the virtual warehouse** (vertical scaling) if the bottleneck is raw compute, or enabling a **multi-cluster warehouse** if the bottleneck is concurrency (many users/queries at once, not one heavy query).
5. Review the SQL itself — unnecessary `SELECT *`, missing filters pushed down, or poorly structured joins.

---

### Q9. Your Snowflake bill suddenly spiked this month. How do you investigate and control it going forward?

1. Check `WAREHOUSE_METERING_HISTORY` / the cost dashboard in Snowsight to see which warehouse(s) consumed the most credits.
2. Check whether **auto-suspend** was set too high (or disabled) on any warehouse — an idle warehouse still burns credits if it doesn't suspend quickly.
3. Look for warehouses that are **oversized** for their actual workload (e.g., a Large warehouse running a query that only needs an Extra-Small).
4. Set up **Resource Monitors** with credit quotas and notification/suspend actions to prevent this from recurring:
   ```sql
   CREATE OR REPLACE RESOURCE MONITOR monthly_limit
   WITH CREDIT_QUOTA = 1000
   TRIGGERS ON 80 PERCENT DO NOTIFY
            ON 100 PERCENT DO SUSPEND;
   ```
5. Longer-term: separate warehouses by workload type (ETL vs. ad-hoc BI queries) so a runaway analyst query doesn't compete with — or get billed against — your production pipeline warehouse.

---

### Q10. A dashboard used by business users runs the same complex aggregation query repeatedly throughout the day. How do you speed it up without changing the underlying tables constantly?

Point to **Materialized Views**: pre-compute and store the result of the expensive aggregation once, and Snowflake automatically keeps it in sync as the base table changes. Queries against the materialized view read the pre-computed result instead of recalculating every time — much faster for **repeated, predictable aggregation queries** on data that doesn't change too frequently (materialized views have refresh/maintenance overhead, so they're not ideal for very high-churn tables).

---

## Section 3: Change Data Capture, Automation & Pipeline Design

### Q11. You need to keep a target table in sync with a source table incrementally — only processing what's new or changed, not the whole table every time. How would you design this in Snowflake?

This is the classic **Streams + Tasks** CDC (Change Data Capture) pattern:
- A **Stream** sits on the source table and automatically records every `INSERT`, `UPDATE`, and `DELETE` since it was last "consumed" — think of it as a change log with metadata columns like `METADATA$ACTION` and `METADATA$ISUPDATE`.
- A **Task** is a scheduled SQL statement (or stored procedure call) that can be configured to run **only when the Stream actually has new data** (`WHEN SYSTEM$STREAM_HAS_DATA(...)`), avoiding wasted runs.
- Together: the Task reads only the changed rows from the Stream and applies them to the target table (often via a `MERGE` statement), then the Stream's offset moves forward automatically.

**A gotcha worth mentioning in an interview:** a Stream goes **stale** if its offset falls outside the source table's data retention period (14 days by default). Once stale, it can no longer report changes and has to be recreated — so long-idle pipelines need monitoring.

---

### Q12. How would you implement Slowly Changing Dimension Type 2 (keep full history of changes, not just the latest value) in Snowflake?

1. Create a **Stream** on the staging/landing table to capture inserts and updates.
2. Build a `MERGE` statement against the dimension table:
   - If the incoming row is **new** (key doesn't exist) → `INSERT` a new record marked `IS_CURRENT = TRUE`.
   - If the incoming row's key **exists but values changed** → `UPDATE` the old record to `IS_CURRENT = FALSE` (close it out with an end date) **and** `INSERT` a fresh row with the new values and `IS_CURRENT = TRUE`.
   - If nothing changed, do nothing.
3. Schedule this `MERGE` with a **Task** that fires whenever the Stream has data.

**Why interviewers ask this:** it checks whether you actually understand dimensional modeling (why we preserve history instead of just overwriting) plus practical Snowflake mechanics (`MERGE`, `Streams`, `Tasks`) — a very common real project requirement, not just theory.

---

### Q13. How do you track *structural* (DDL) changes to your tables — like a teammate silently adding, dropping, or changing the type of a column — since Snowflake doesn't have a built-in "DDL history" feature the way it has Time Travel for data?

Build a simple **snapshot-comparison ("diff") pattern**:
1. Keep a `CURRENT_SNAPSHOT` table that stores the current list of tables/columns/data types (pulled from `INFORMATION_SCHEMA.COLUMNS`) — this is your **"before" picture**.
2. In a scheduled stored procedure, compare that snapshot against the **live** `INFORMATION_SCHEMA.COLUMNS` (the **"after" picture**):
   - Column in "after" but not "before" → **ADDED**.
   - Column in "before" but not "after" → **DROPPED**.
   - Column in both, but data type differs → **MODIFIED**.
3. Log all differences (who changed what, when) into a `DDL_CHANGE_LOG` audit table.
4. Refresh the snapshot at the end of each run so the next comparison starts clean.

This is a genuinely popular scenario question because it tests **`LEFT JOIN` logic** (used to detect "missing on one side") more than it tests raw Snowflake trivia.

---

### Q14. Your company needs to give an external partner secure, read-only access to specific sensitive data — without physically copying or emailing files, and while still masking sensitive fields. How would you architect this?

- Use **Secure Data Sharing** so the partner queries live data directly through their own Snowflake account (or a Reader Account if they don't have Snowflake) — no data is copied or moved.
- Wrap the shared data behind a **Secure View** so the partner can only see the specific columns/rows you expose, not the full underlying table.
- Apply **Dynamic Data Masking** on sensitive columns (e.g., SSNs, account numbers) so masking rules apply automatically based on the querying role.
- Apply **Row Access Policies** if different partners should see different row-level subsets of the same table.
- Enforce all of this through **RBAC** — the partner's role should be scoped to only what they need (principle of least privilege).

---

### Q15. Your team wants both a testing/QA environment and a production environment, but copying terabytes of production data every time for testing is too slow and expensive. What's the Snowflake-native solution?

**Zero-Copy Cloning.** A clone of a database, schema, or table is created **instantly** and doesn't physically duplicate the underlying data — it just creates new metadata pointers to the same micro-partitions. Storage cost is only incurred for data that **changes** after the clone is made (since Snowflake then has to store the new/changed micro-partitions separately). This makes it ideal for spinning up realistic QA/test environments from production data without doubling storage costs or waiting hours for a copy job.

```sql
CREATE OR REPLACE DATABASE QA_DB CLONE PROD_DB;
```

---

### Q16. Someone accidentally ran an `UPDATE` or `DELETE` without a `WHERE` clause on a production table. How do you recover the data?

Use **Time Travel** to query or restore the table as it existed *before* the mistake:
```sql
-- View the data as it was 10 minutes ago
SELECT * FROM my_table AT(OFFSET => -60*10);

-- Or restore the entire table to a point before the mistake
CREATE OR REPLACE TABLE my_table_restored CLONE my_table
  AT (OFFSET => -60*10);

-- Or, if the table itself was dropped:
UNDROP TABLE my_table;
```
Time Travel retention is configurable (up to 90 days depending on edition), so mention that the recovery window depends on the account's `DATA_RETENTION_TIME_IN_DAYS` setting for that table/database.

---

## Section 4: Quick-Fire Situational Questions (commonly seen on LinkedIn / interview-prep posts)

| Situation | Expected Answer (short) |
|---|---|
| Same file loaded twice by mistake, no `FORCE=TRUE` used | Snowflake automatically skips it — no duplicate rows, thanks to load metadata tracking |
| Need to reload files older than the usual load-history window | Manually re-stage the file (which changes its checksum) or use `FORCE = TRUE`, being aware of possible duplication |
| Want a warehouse to save cost during idle periods | Configure a short `AUTO_SUSPEND` and `AUTO_RESUME = TRUE` so compute isn't billed while idle |
| Many concurrent BI users are causing queuing on one warehouse | Convert it to a **multi-cluster warehouse** so it scales horizontally to serve more concurrent queries |
| Need to store and query nested JSON from an API | Load into a `VARIANT` column and query it directly using dot notation / `FLATTEN()` — no rigid schema needed upfront |
| Need to query data sitting in S3 without physically loading it into Snowflake | Create an **External Table** over the S3 location using a storage integration and file format |
| A pipeline should only re-run downstream steps when upstream data actually changed | Use a **Task** with a `WHEN SYSTEM$STREAM_HAS_DATA(...)` condition, or chain Tasks into a **Task Tree**, rather than a blind time-based schedule |

---

## 📌 Key Takeaway for Interview Prep

Nearly all of these "real-time" scenario questions boil down to a small set of core Snowflake building blocks used in combination:

- **`COPY INTO` + `ON_ERROR` + `VALIDATE()`** → resilient data loading
- **Stored Procedures + `EXECUTE IMMEDIATE`** → dynamic, reusable automation
- **`INFER_SCHEMA` + `ENABLE_SCHEMA_EVOLUTION`** → handling changing file structures
- **Streams + Tasks** → incremental/CDC pipelines and SCD Type 2
- **Time Travel + Zero-Copy Cloning** → data recovery and cheap test environments
- **Resource Monitors + warehouse sizing** → cost control
- **RBAC + Secure Views + Data Masking** → secure data sharing

If you can explain **why** you'd reach for each of these (not just what they do), you can handle almost any scenario-based Snowflake interview question, even ones phrased in an unfamiliar way.

---

*Sources referenced while compiling this guide: Data Engineer Academy, Snowflake official documentation, phData, ThinkETL, Medium engineering write-ups (Kiran Mai Malluvalasa, Amit Yadav, Viraj Chavan), CelestInfo, and various Snowflake-focused LinkedIn/interview-prep blogs. Content has been paraphrased and reorganized into an original Q&A format — refer to the original sources directly for their full write-ups.*


# Snowflake Interview Q&A — Concept, Question, Answer, Solution

> Compiled and cleaned up from 29 "Pravin Kumar" YouTube transcripts on Snowflake/SQL interview preparation.
> Organized by concept area. Each entry follows: **Concept → Question → Answer → Solution (syntax/example, where applicable)**.

---

## 1. Views (Normal, Secure, Materialized, Temporary)

**Concept:** Snowflake supports four kinds of views — normal (non-materialized), secure, materialized, and temporary — and each behaves differently for storage and visibility of the view definition.

**Q1. Does a materialized view have physical storage?**
**A:** Yes. A materialized view is a *pre-computed, pre-calculated result set that is physically stored*, which is why querying it is faster. A normal (non-materialized) view has **no** storage of its own — it just re-runs its underlying query each time.
**Solution:**
```sql
-- Check storage for views
SELECT * FROM information_schema.table_storage_metrics
WHERE table_name IN ('EMP_V', 'EMP_MV');

-- Or
SHOW MATERIALIZED VIEWS;
```
Only the materialized view row shows non-zero `ACTIVE_BYTES`; the normal view shows no entry/zero.

**Q2. Can a materialized view be created on multiple tables (a join)?**
**A:** No. A materialized view can be created on **only a single table**. If you need to combine multiple tables, use a **dynamic table** instead.

**Q3. Can you perform DML on a materialized view?**
**A:** No, DML operations (INSERT/UPDATE/DELETE) are not allowed directly on a materialized view.

**Q4. How do you convert a normal view into a secure view, and back?**
**A:** Use the `SECURE` keyword when creating, or `ALTER VIEW ... SET SECURE` / `UNSET SECURE` on an existing view.
**Solution:**
```sql
-- Create secure view directly
CREATE SECURE VIEW emp_v AS SELECT * FROM emp;

-- Convert normal view -> secure view
ALTER VIEW emp_v SET SECURE;

-- Convert secure view -> normal view
ALTER VIEW emp_v UNSET SECURE;

-- Check status
SHOW VIEWS;   -- look at the "is_secure" column (true/false)
```

**Q5. What is a secure view, and does it have storage?**
**A:** A secure view hides the view's *definition* (SQL logic) from users who only have access to query it — they can see the data but not how it was derived. Like a normal view, a secure view has **no physical storage**; only materialized views store data.

**Q6. How do you create a temporary view?**
**A:** Use the `TEMPORARY` keyword. A temporary view exists only for the current session.
**Solution:**
```sql
CREATE TEMPORARY VIEW emp_temp_v AS
SELECT * FROM emp;
```

---

## 2. Cloning (Zero-Copy Cloning)

**Concept:** Cloning in Snowflake ("zero-copy cloning") creates a new object (table/schema/database) that shares the same **micro-partition metadata** as the source, without physically copying the data. It's used mainly to take backups or spin up test/dev copies cheaply.

**Q1. Does a cloned object have its own storage?**
**A:** No — at the moment of cloning, no additional storage is used. Only the metadata pointing to the same micro-partitions is copied. Storage is only consumed later if/when the clone or the source diverges (new inserts/updates/deletes create new micro-partitions for whichever object changed).

**Q2. What exactly happens in the background during a clone?**
**A:** Data is physically stored in *micro-partitions* in the storage layer. Cloning copies only the **metadata** that maps a table to its micro-partitions — not the underlying bytes. That's why it's called zero-copy.
**Solution:**
```sql
CREATE TABLE emp_clone CLONE emp;
CREATE DATABASE new_db CLONE old_db;
```

**Q3. How do you tell whether a table is a normal table or a cloned table?**
**A:** Compare the `ID` and `CLONE_GROUP_ID` columns in `information_schema.table_storage_metrics`. If they're the same, it's a normal (base) table. If they differ, it's a clone (and the group ID tells you which base table it was cloned from).
**Solution:**
```sql
SELECT id, clone_group_id, table_name
FROM information_schema.table_storage_metrics
WHERE table_catalog = current_database();
```

**Q4. If I update the clone, does it affect the base table (or vice versa)?**
**A:** No. Once cloned, the base table and the clone are **completely independent**. Changes to one do not reflect in the other, and each will start accumulating its own separate storage cost for the new/changed data.

**Q5. Can you clone an external table?**
**A:** No — external tables cannot be cloned (they only hold file metadata, not actual data, so there's nothing physical to snapshot).

**Q6. Zero-copy cloning vs. Time Travel for taking a backup — which is preferred and why?**
**A:** Cloning is preferred for creating a full backup/dev copy because it has **no storage cost and no compute cost** at creation time — it's a metadata-only "replication."

---

## 3. Time Travel

**Concept:** Time Travel lets you query/restore historical versions of data (before an UPDATE/DELETE/TRUNCATE/DROP) using a **query ID**, an **offset** (e.g., minus N minutes), or a **timestamp**.

**Q1. What are the ways to access historical data via Time Travel?**
**A:** Three ways — by `QUERY ID`, by `OFFSET`, or by `TIMESTAMP`.
**Solution:**
```sql
-- By query ID
SELECT * FROM emp BEFORE (STATEMENT => '<query_id>');

-- By offset (seconds)
SELECT * FROM emp AT (OFFSET => -60*5);   -- 5 minutes ago

-- By timestamp
SELECT * FROM emp AT (TIMESTAMP => '2026-06-01 10:00:00'::timestamp);
```

**Q2. What is the default retention time for Time Travel?**
**A:** **1 day** by default for permanent, transient, and temporary tables.

**Q3. What's the maximum retention time you can configure, and how?**
**A:** Up to **90 days** for permanent tables (Enterprise edition and above); use `ALTER TABLE ... SET DATA_RETENTION_TIME_IN_DAYS`.
**Solution:**
```sql
ALTER TABLE emp SET DATA_RETENTION_TIME_IN_DAYS = 90;
```

**Q4. How do you disable Time Travel for a table?**
**A:** Set the retention time to `0`.
**Solution:**
```sql
ALTER TABLE emp SET DATA_RETENTION_TIME_IN_DAYS = 0;
```

**Q5. A table was accidentally dropped — how do you recover it?**
**A:** Use `UNDROP TABLE`, which relies on Time Travel.
**Solution:**
```sql
UNDROP TABLE emp;
```

**Q6. What if `UNDROP` fails because an object with the same name already exists (e.g., you recreated the table after dropping it)?**
**A:** First rename the existing (new) object to free up the name, then run `UNDROP` for the old one to bring it back with its original name.
**Solution:**
```sql
ALTER TABLE emp RENAME TO emp_bkp;
UNDROP TABLE emp;
```

**Q7. How far back can Query History be viewed from the UI vs. via SQL?**
**A:** From the Snowsight/Classic Console UI's Query History screen, only the **last 14 days**. Using `snowflake.account_usage.query_history`, you can go back up to **365 days**.
**Solution:**
```sql
SELECT * FROM snowflake.account_usage.query_history
WHERE query_text ILIKE '%TRUNCATE TABLE%'
ORDER BY start_time DESC;
```

**Q8. Can Time Travel be used on an external table?**
**A:** No — Time Travel is not available for external tables (they don't store data physically in Snowflake).

**Q9. Does `CREATE OR REPLACE TABLE` let you Time Travel back after it runs?**
**A:** No — when you `CREATE OR REPLACE` a table, the old table is dropped and a brand-new table object is created. Time Travel on the *new* table won't show the old table's data since it's a different table object; you'd have to `UNDROP` the old (dropped) version instead — after renaming the new one out of the way if needed.

---

## 4. Fail-safe

**Concept:** Fail-safe is a 7-day (fixed) recovery period that begins **after** Time Travel retention ends, available only for **permanent tables**. It exists purely as a last-resort recovery mechanism controlled by Snowflake support, not the customer.

**Q1. Can you configure or disable Fail-safe?**
**A:** No — Fail-safe cannot be configured, extended, reduced, or disabled. If you don't want Fail-safe overhead at all, use a **transient** or **temporary** table (these have no Fail-safe).

**Q2. Which table types get Fail-safe?**
**A:** Only **permanent tables**. Transient and temporary tables have **zero** Fail-safe period.

**Q3. Can a Snowflake developer/customer directly query or restore data from Fail-safe?**
**A:** No. Fail-safe data is not directly accessible — you must raise a support/service ticket with Snowflake, and their support team (not the customer) performs the recovery, typically within about 7 days.

**Q4. Is Fail-safe storage counted toward the customer's storage cost?**
**A:** Yes — data sitting in Fail-safe still contributes to storage cost/billing.

**Q5. Is Fail-safe a reliable way to set up Dev/Test environments?**
**A:** No — Fail-safe is meant only as an emergency recovery mechanism for **production** data, not a mechanism to spin up Dev/Test copies.

---

## 5. Table Types: Permanent, Transient, Temporary, External

**Concept:** Snowflake tables are broadly *internal* (data physically stored inside Snowflake as micro-partitions) or *external* (only file metadata is tracked; actual files live in S3/Azure/GCS). Internal tables are further split into permanent, transient, and temporary.

**Q1. What table types does Snowflake support?**
**A:** Internal tables: **Permanent**, **Transient**, **Temporary**. Plus **External tables** (data lives outside Snowflake; Snowflake only stores file name/row metadata).

**Q2. What table type is created by default?**
**A:** **Permanent table** — unless you explicitly specify `TRANSIENT` or `TEMPORARY`/`TEMP`.
**Solution:**
```sql
CREATE TABLE emp (...);              -- permanent (default)
CREATE TRANSIENT TABLE emp_t (...);  -- transient
CREATE TEMPORARY TABLE emp_tmp (...);-- temporary
```

**Q3. What's the difference between Permanent, Transient, and Temporary tables?**
**A:**
| Aspect | Permanent | Transient | Temporary |
|---|---|---|---|
| Fail-safe | Yes (7 days) | No | No |
| Time Travel (default) | 1 day (up to 90) | 1 day (up to 90 for Transient too, but typically kept low) | 0–1 day |
| Scope | Persists normally | Persists normally | **Session-specific** — auto-dropped when session ends |

**Q4. If I create a table inside a *transient* database or schema, what type is the table by default?**
**A:** Even without specifying `TRANSIENT` on the table itself, if the parent database or schema is transient, the table created inside it is automatically **transient**. (Table type is inherited from a transient database/schema.)

**Q5. What can you perform on external tables?**
**A:** Only read/query access — you **cannot perform DML** on external tables, because the actual data isn't stored in Snowflake; it just references files in cloud storage.

**Q6. Is TRUNCATE a DDL or DML command in Snowflake?**
**A:** In Snowflake, `TRUNCATE` is treated as a **DML** command (unlike most traditional RDBMS where it's DDL).

**Q7. Are DML statements auto-committed in Snowflake?**
**A:** Yes — DML in Snowflake is **auto-committed**; there's no need for an explicit `COMMIT`.

---

## 6. Stages (Internal / External)

**Concept:** Stages are temporary storage locations used to hold files before loading (or after unloading) data. Internal stages keep files inside Snowflake; external stages point to files in cloud storage (S3/Azure/GCS).

**Q1. What are the types of internal stages?**
**A:** **User stage**, **Table stage**, and **Named internal stage**.
- User stage: auto-created for every user, referenced as `@~`
- Table stage: auto-created for every table, referenced as `@%<table_name>`
- Named stage: explicitly created, referenced as `@<stage_name>`

**Q2. How do you list files in each stage type?**
**Solution:**
```sql
LIST @~;                 -- user stage
LIST @%emp;               -- table stage (for table 'emp')
LIST @my_named_stage;     -- named internal stage
```

**Q3. How do you create a named internal stage?**
**Solution:**
```sql
CREATE STAGE my_named_stage;
```

**Q4. How do you find whether a stage is internal or external?**
**Solution:**
```sql
SHOW STAGES;   -- check the "type" column
```

**Q5. How do you remove a file from a stage?**
**Solution:**
```sql
REMOVE @my_named_stage/filename.csv;
-- or the alias
RM @my_named_stage/filename.csv;
```

**Q6. How do you read a specific column from a staged file directly, or get the file name?**
**Solution:**
```sql
SELECT $1, $5 FROM @my_named_stage;               -- 1st and 5th columns
SELECT metadata$filename FROM @my_named_stage;    -- file name
```

**Q7. Do `PUT` and `GET` commands work through the Snowsight/Classic Console UI?**
**A:** No — `PUT` (upload local file → internal stage) and `GET` (download from internal stage → local machine) work **only through SnowSQL (CLI)**, not the browser UI.

**Q8. What's the maximum file size you can load directly through the UI (Snowsight/Classic Console)?**
**A:** Up to **250 MB** per file via UI upload. For larger files, use SnowSQL or an external stage with `COPY INTO`.

---

## 7. Snowpipe

**Concept:** Snowpipe provides continuous, near-real-time, automated data ingestion from a stage into a table using `COPY INTO` under the hood, without manual intervention — best suited for continuously arriving files.

**Q1. Can Snowpipe be created on internal stages, or only external stages?**
**A:** Snowpipe can be created on **both** — but for internal stages specifically, it works on the **table stage** and **named internal stage** only. It **cannot** be created on the **user stage**.
**Solution:**
```sql
-- Works (table stage)
CREATE OR REPLACE PIPE emp_pipe
  AUTO_INGEST = FALSE
  AS COPY INTO emp FROM @%emp;

-- Works (named internal stage)
CREATE OR REPLACE PIPE emp_pipe2
  AUTO_INGEST = FALSE
  AS COPY INTO emp FROM @my_named_stage;

-- Fails (user stage) -> "Stage cannot be used ..." compilation error
```

**Q2. What is the syntax to create a Snowpipe?**
**Solution:**
```sql
CREATE OR REPLACE PIPE pipe_name
  AUTO_INGEST = TRUE
  AS COPY INTO target_table
  FROM @stage_name;
```

**Q3. What is the default status of a newly created Snowpipe?**
**A:** A newly created pipe is in a **running** state by default (unlike Tasks, which default to *suspended*).

**Q4. How do you check the status of a Snowpipe?**
**Solution:**
```sql
SELECT SYSTEM$PIPE_STATUS('pipe_name');
```

**Q5. How do you refresh a Snowpipe manually?**
**Solution:**
```sql
ALTER PIPE pipe_name REFRESH;
```

**Q6. How do you pause/resume Snowpipe (e.g., after testing)?**
**Solution:**
```sql
ALTER PIPE pipe_name SET PIPE_EXECUTION_PAUSED = TRUE;   -- pause
ALTER PIPE pipe_name SET PIPE_EXECUTION_PAUSED = FALSE;  -- resume
```

**Q7. What's the recommended file size for Snowpipe loads?**
**A:** Roughly **100 MB to 250 MB** (compressed) per file for optimal performance.

**Q8. How long does Snowpipe retain load-history metadata (to avoid re-loading the same file)?**
**A:** Up to **14 days** (compare: `COPY INTO` retains load metadata for **64 days**).

---

## 8. Streams (Change Data Capture)

**Concept:** A Stream tracks row-level changes (insert/update/delete) on a table since it was last consumed — the mechanism used to implement CDC (Change Data Capture) in Snowflake.

**Q1. What is the syntax to create a stream?**
**Solution:**
```sql
CREATE OR REPLACE STREAM my_stream ON TABLE emp;
```

**Q2. What are the metadata columns a stream adds?**
**A:** Three key metadata columns:
- `METADATA$ACTION` — INSERT or DELETE
- `METADATA$ISUPDATE` — boolean, true if the row change is part of an UPDATE
- `METADATA$ROW_ID` — unique row identifier

**Q3. How do you check whether a stream has unconsumed data?**
**Solution:**
```sql
SELECT SYSTEM$STREAM_HAS_DATA('my_stream');   -- returns TRUE/FALSE
```

**Q4. Can DML be performed directly on a stream object?**
**A:** No — streams are read-only change logs; you cannot INSERT/UPDATE/DELETE on a stream itself.

**Q5. Can you create multiple streams on a single table?**
**A:** Yes — multiple independent streams can be created on the same source table.

**Q6. Can a stream be cloned?**
**A:** Yes, stream objects can be cloned.

**Q7. What types of streams are available?**
**A:** **Standard stream** (default — captures inserts, updates, deletes), and **Append-only / Insert-only stream** (captures only new inserted rows — commonly used with external tables or when you only care about new data).

---

## 9. Tasks (Scheduling / Orchestration)

**Concept:** Tasks are Snowflake's built-in scheduler, used to run a SQL statement or call a stored procedure on a schedule, optionally chained together via dependencies.

**Q1. What is the default status of a newly created task?**
**A:** **Suspended.** You must explicitly resume it.
**Solution:**
```sql
ALTER TASK task_name RESUME;
```

**Q2. What's the syntax to create a task, e.g., scheduled daily at 1:00 AM?**
**Solution:**
```sql
CREATE TASK my_task
  WAREHOUSE = compute_wh
  SCHEDULE = 'USING CRON 0 1 * * * UTC'
AS
  <SQL statement or CALL procedure>;
```

**Q3. How do you make one task run only after another task completes (task dependency/chaining)?**
**A:** Use the `AFTER` keyword to define dependency.
**Solution:**
```sql
CREATE TASK silver_to_gold_task
  WAREHOUSE = compute_wh
  AFTER bronze_to_silver_task
AS
  <SQL statement>;
```

**Q4. How is orchestration typically implemented in Snowflake?**
**A:** For orchestration **within** Snowflake, use the built-in **Task** feature. For orchestration involving **external systems/dependencies**, companies commonly use **Apache Airflow** or **Azure Data Factory**.

---

## 10. Data Loading & Unloading

**Concept:** Loading data into Snowflake is typically done via `COPY INTO` (bulk/batch), Snowpipe (continuous), or querying external tables directly. Error handling during load is controlled by the `ON_ERROR` option.

**Q1. How do you load only the valid rows from a file, skipping bad rows?**
**A:** Use `ON_ERROR = CONTINUE`.
**Solution:**
```sql
COPY INTO emp
FROM @my_stage
ON_ERROR = 'CONTINUE';
```

**Q2. How do you make the load fail completely (load nothing) if even one row errors?**
**A:** Use `ON_ERROR = ABORT_STATEMENT` (this is also the default behavior).
**Solution:**
```sql
COPY INTO emp
FROM @my_stage
ON_ERROR = 'ABORT_STATEMENT';
```

**Q3. What function helps identify data load errors without actually loading the data?**
**A:** The `VALIDATE` function.
**Solution:**
```sql
SELECT * FROM TABLE(VALIDATE(emp, JOB_ID => '<copy_job_id>'));
```

**Q4. What is the file size limit for data unloading (`COPY INTO` a stage/location) per file?**
**A:** **5 GB** per file (there's no cap on total data unloaded, only per-file).

**Q5. How long does `COPY INTO` retain load metadata (to prevent duplicate loads of the same file)?**
**A:** **64 days.**

**Q6. How does Snowflake handle schema drift (source file structure changes, e.g., new columns appear)?**
**A:** Enable **schema evolution** on the table/pipe so new columns are automatically adapted without manual DDL changes.

**Q7. How do you handle nested/semi-structured JSON or arrays to extract individual values?**
**A:** Use `LATERAL FLATTEN` to explode the nested structure into rows/columns.
**Solution:**
```sql
SELECT f.value:column_name::string
FROM my_table, LATERAL FLATTEN(input => my_table.json_column) f;
```

**Q8. How do you store semi-structured data (JSON, XML, Avro, Parquet, ORC) in a Snowflake table?**
**A:** Use the **`VARIANT`** data type.

---

## 11. Dynamic Data Masking & Row-Level Security

**Concept:** Dynamic Data Masking hides sensitive column values based on the querying user's **role**, unlike static masking, which hides data for everyone regardless of role.

**Q1. What's the difference between static masking and dynamic masking?**
**A:** Static masking hides the sensitive value for **all** users equally. Dynamic masking evaluates the **current role** at query time and shows full value to authorized roles while masking it for others.

**Q2. How do you create and apply a masking policy?**
**Solution:**
```sql
CREATE OR REPLACE MASKING POLICY other_mask AS (val STRING)
RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() = 'DATA_ADMIN' THEN val
    ELSE CONCAT(REPEAT('X', LENGTH(val) - 4), SUBSTR(val, -4))
  END;

ALTER TABLE emp MODIFY COLUMN phone_number
  SET MASKING POLICY other_mask;
```

**Q3. Scenario: business users should see masked salary/email/phone, but HR should see full values — what feature do you use?**
**A:** Dynamic data masking combined with **RBAC (role-based access control)** — assign a masking policy that checks `CURRENT_ROLE()`.

**Q4. Scenario: a company wants users to only see rows where `department = 'Sales'` — what feature achieves row-level filtering?**
**A:** **Row Access Policy.**
**Solution:**
```sql
CREATE ROW ACCESS POLICY sales_policy
AS (department STRING) RETURNS BOOLEAN ->
  department = 'SALES';

ALTER TABLE emp ADD ROW ACCESS POLICY sales_policy ON (department);
```

---

## 12. Snowflake Architecture

**Concept:** Snowflake has a 3-layer architecture that separates storage from compute — this separation is the core design that enables independent scaling of storage and compute.

**Q1. What are the three layers of Snowflake's architecture?**
**A:**
1. **Cloud Services Layer** — authentication, authorization, metadata management, query optimization, access control.
2. **Query Processing / Compute Layer** — virtual warehouses that provide CPU/RAM to execute DML, DRL, procedures, data loading/unloading.
3. **Database Storage Layer** — actual data stored physically in **micro-partitions** in cloud storage (S3/Azure Blob/GCS).

**Q2. When you run a DDL statement (e.g., `CREATE TABLE`), which layer is involved, and does the warehouse need to be running?**
**A:** Only the **Cloud Services layer** (metadata management) is involved. DDL doesn't require compute, so the virtual warehouse can remain **suspended**.

**Q3. When you run a DML statement (e.g., `INSERT`), what happens?**
**A:** The virtual warehouse is required (it auto-resumes if suspended) to physically write the data into micro-partitions in the storage layer; the Cloud Services layer records the metadata about which micro-partitions hold that data.

**Q4. What model does Snowflake follow commercially?**
**A:** **SaaS (Software as a Service)** and **pay-as-you-go** — you pay based on storage used and compute (warehouse) usage, no software/hardware installation required.

---

## 13. Constraints

**Concept:** Snowflake supports declaring standard relational constraints, but for performance reasons only **enforces** one of them at runtime.

**Q1. Which constraints does Snowflake support, and which does it actually enforce?**
**A:** Snowflake **supports** (allows you to declare) `UNIQUE`, `NOT NULL`, `PRIMARY KEY`, `FOREIGN KEY`, and `DEFAULT` constraints — but it only **enforces** the `NOT NULL` constraint. The others are informational/metadata only.

**Q2. Can a Primary Key or Unique column have duplicate values in Snowflake?**
**A:** Yes — because only `NOT NULL` is enforced, Snowflake will **allow duplicate values** even in columns declared `PRIMARY KEY` or `UNIQUE`. (It just won't allow NULLs in a Primary Key column.)

**Q3. How many primary keys can a single table have?**
**A:** Only **one** (can be a composite key spanning multiple columns if needed).

**Q4. How do you establish a relationship between two tables?**
**A:** Using a **Foreign Key** (referential integrity constraint) — even though Snowflake doesn't enforce it, it's still used for documentation/metadata/BI-tool relationship discovery.

---

## 14. Caching

**Concept:** Snowflake uses multiple cache layers to speed up repeated queries and avoid recomputation.

**Q1. What types of caching does Snowflake use?**
**A:** **Result cache**, **local disk (warehouse) cache**, and **metadata cache**.

**Q2. How long does the result cache persist by default, and can it be extended?**
**A:** **24 hours (1 day)** by default; can be extended up to **31 days**.

**Q3. What conditions invalidate the result cache?**
**A:** If the underlying data changes, or the query text changes, the cached result is no longer used.

---

## 15. SQL Joins — Predicting Record Counts

**Concept:** A very common interview drill: given the count of matching/non-matching keys between two tables, predict the row counts returned by INNER, LEFT, RIGHT, FULL, and CROSS joins.

**Q1. What's the quick logic to calculate join output row counts?**
**A:**
- **INNER JOIN** = matching (common) records between the two tables.
- **LEFT JOIN** = INNER JOIN result + remaining (non-matching) records from the **left** table.
- **RIGHT JOIN** = INNER JOIN result + remaining (non-matching) records from the **right** table.
- **FULL JOIN** = INNER JOIN result + remaining records from **both** left and right tables.
- **CROSS JOIN** = (row count of table 1) × (row count of table 2).

**Solution — worked example:** Left table has three `1`s and two `2`s; right table has two `1`s and three `2`s.
```
INNER = (3×2) + (2×3) = 6 + 6 = 12
LEFT  = 12 + (unmatched left rows)
RIGHT = 12 + (unmatched right rows)
FULL  = 12 + unmatched left + unmatched right
CROSS = (total left rows) × (total right rows)
```

**Q2. Do NULLs match each other in a join, e.g., in INNER JOIN?**
**A:** No — `NULL` can never be equal to another `NULL` (comparing NULL to anything, including NULL, is unknown/false). So **INNER JOIN never returns matched NULL rows** — NULLs only show up in LEFT/RIGHT/FULL JOIN as *unmatched* rows, and they *do* count as rows in a CROSS JOIN.

**Q3. `SELECT 1 UNION SELECT 1` — what's the output?**
**A:** A single row with value `1` — `UNION` removes duplicates.

**Q4. `SELECT 1 UNION ALL SELECT 1` — what's the output?**
**A:** Two rows, both `1` — `UNION ALL` keeps duplicates (and is faster than `UNION` since it skips the de-duplication step).

**Q5. `SELECT NULL` — what's the output?**
**A:** `NULL` — any operation/expression involving `NULL` (unless specifically handled, e.g., `COALESCE`) evaluates to `NULL`.

**Q6. `CASE WHEN NULL = NULL THEN ... END` with no `ELSE` — what happens?**
**A:** No syntax error — the condition simply evaluates to `FALSE`/unknown, and since there's no `ELSE`, the result is `NULL`.

**Q7. `EMP CROSS JOIN EMP1 WHERE 1 = 1` — what kind of join is this really?**
**A:** It's effectively still a **CROSS JOIN**, because `1 = 1` is always true and has no real join predicate linking the two tables — so it multiplies row counts just like an explicit `CROSS JOIN`.

---

## 16. Window / Analytical Functions

**Concept:** `ROW_NUMBER`, `RANK`, and `DENSE_RANK` are the go-to analytical functions for "Nth highest/lowest" style questions, with `QUALIFY` used to filter directly on them.

**Q1. Difference between `ROW_NUMBER`, `RANK`, and `DENSE_RANK`?**
**A:**
- `ROW_NUMBER()` — assigns a unique sequential number to every row, even if values (e.g., salary) are duplicated.
- `RANK()` — gives the same rank to duplicate values, but then **skips** the next rank number(s) accordingly.
- `DENSE_RANK()` — gives the same rank to duplicate values but does **not skip** the next rank number.

**Q2. If there are no duplicate values, which function should you use for "Nth highest/lowest"?**
**A:** Any of the three works the same when there are no duplicates — typically `ROW_NUMBER()` is used for simplicity.

**Q3. If duplicate values exist, which should you use?**
**A:** `DENSE_RANK()` — because it doesn't skip ranks, so "Nth highest" reflects the Nth **distinct** value correctly.

**Q4. How do you get the Nth lowest (or highest) salary?**
**Solution:**
```sql
SELECT *
FROM (
  SELECT e.*, DENSE_RANK() OVER (ORDER BY salary ASC) AS dr
  FROM emp e
)
WHERE dr = 50;   -- 50th lowest; use DESC for highest
```
Or with `QUALIFY` (Snowflake-specific, avoids the extra subquery):
```sql
SELECT *, DENSE_RANK() OVER (ORDER BY salary ASC) AS dr
FROM emp
QUALIFY dr = 50;
```

**Q5. How do you get the Nth highest salary *per department*?**
**Solution:**
```sql
SELECT *, DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dr
FROM emp
QUALIFY dr = 4;
```

**Q6. How do you get the top 10 salaries in each department?**
**Solution:**
```sql
SELECT *, DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dr
FROM emp
QUALIFY dr <= 10;
```

**Q7. What does `QUALIFY` do, and how is it different from `WHERE`/`HAVING`?**
**A:** `QUALIFY` filters on the result of a **window/analytical function**, which `WHERE` cannot do directly (since window functions are evaluated after `WHERE`). `WHERE` filters raw table rows; `HAVING` filters `GROUP BY` aggregated groups; `QUALIFY` filters analytical function results.

---

## 17. String Functions

**Concept:** Common string manipulation asked in interviews: extracting characters, finding substrings, counting occurrences, and reversing delimited lists.

**Q1. How do you check if a string starts and ends with the same character?**
**A:** Use `SUBSTR` to grab the first and last character and compare them.
**Solution:**
```sql
SELECT name
FROM emp
WHERE SUBSTR(name, 1, 1) = SUBSTR(name, -1, 1);
```

**Q2. How do you find the position of a character in a string?**
**A:** Use `POSITION` / `INSTR` (Snowflake also supports specifying the Nth occurrence).
**Solution:**
```sql
SELECT POSITION('e' IN name);
SELECT INSTR(name, 'e', 1, 2);  -- 2nd occurrence of 'e'
```

**Q3. How do you count the number of occurrences of a character in a string?**
**A:** Preferred: `REGEXP_COUNT`. Without regex: subtract the length of the string after removing the character from the original length.
**Solution:**
```sql
SELECT REGEXP_COUNT(name, 'e');

-- without regex
SELECT LENGTH(name) - LENGTH(REPLACE(name, 'e', ''));
```

**Q4. How do you reverse the order of a comma/hyphen-separated list produced by aggregation?**
**A:** Use `LISTAGG` with `WITHIN GROUP (ORDER BY ... DESC)`.
**Solution:**
```sql
SELECT LISTAGG(col, '-') WITHIN GROUP (ORDER BY col DESC)
FROM my_table;
```

**Q5. How do you "swap" values, e.g., turn all `1`s into `0`s and all `0`s into `1`s in a column?**
**A:** Use a `CASE` expression (or arithmetic trick like `1 - column` for binary 0/1 values).
**Solution:**
```sql
SELECT CASE WHEN flag = 1 THEN 0 ELSE 1 END AS swapped_flag FROM my_table;
-- or, purely arithmetic for 0/1 values:
SELECT 1 - flag AS swapped_flag FROM my_table;
```

---

## 18. Auditing & Query History

**Concept:** `information_schema.query_history` (and `snowflake.account_usage.query_history` for longer retention) lets you audit who ran what.

**Q1. How do you find who dropped a specific table?**
**Solution:**
```sql
SELECT user_name, query_text, start_time
FROM information_schema.query_history()
WHERE query_text ILIKE '%DROP TABLE CUSTOMER%'
ORDER BY start_time DESC;
```

**Q2. How far back does `information_schema.query_history` go vs. `snowflake.account_usage.query_history`?**
**A:** `information_schema.query_history` (via the UI/function) covers about **14 days**; `snowflake.account_usage.query_history` covers up to **365 days**.

---

## 19. Data Sharing

**Concept:** Snowflake enables secure, live data sharing between accounts without physically copying data.

**Q1. How do you share data with another Snowflake account (Direct Share)?**
**A:** If the consumer already has a Snowflake account, you can use **Direct Share** to share a database/table/view without copying it.

**Q2. How do you share data with a partner who does *not* have a Snowflake account?**
**A:** Create a **Reader Account** for them — Snowflake provisions a lightweight account (with its own username/password/URL) that lets them query the shared data without needing a full Snowflake subscription of their own.

---

## 20. Snowflake General / Editions / Roles / System Info

**Concept:** Miscellaneous, frequently repeated "warm-up" interview questions on basic system/session commands and editions.

| Question | Answer / Solution |
|---|---|
| Ways to log into Snowflake | Classic Console, Snowsight (UI), and SnowSQL (CLI) |
| Check current Snowflake version | `SELECT CURRENT_VERSION();` |
| Snowflake editions | Standard, Enterprise, Business Critical, Virtual Private Snowflake (VPS) |
| Clouds supported | AWS, Microsoft Azure, GCP (multi-cloud) |
| Check current database/schema/user/role/warehouse | `SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_USER(), CURRENT_ROLE(), CURRENT_WAREHOUSE();` |
| Check current region (cloud) | `SELECT CURRENT_REGION();` |
| Check today's date/time | `SELECT CURRENT_DATE(), CURRENT_TIME();` |
| List tables in current schema | `SHOW TABLES;` or `SELECT * FROM information_schema.tables;` |
| List columns of a table | `DESCRIBE TABLE table_name;` or `SHOW COLUMNS IN table_name;` |
| Get a table's DDL (CREATE statement) | `SELECT GET_DDL('table', 'table_name');` |
| Built-in vs. user-defined functions | `SHOW FUNCTIONS;` (built-in + UDF) vs. `SHOW USER FUNCTIONS;` (UDF only) |
| How to call a UDF | `SELECT function_name(args);` |
| How to call a stored procedure | `CALL procedure_name(args);` |
| Suspend a virtual warehouse | `ALTER WAREHOUSE wh_name SUSPEND;` |
| Vertical scaling | Increasing a single virtual warehouse's **size** (e.g., XS → S → M → L) |
| Horizontal scaling | Increasing the **number of clusters** in a multi-cluster warehouse |
| Query pruning | Skipping/eliminating unnecessary micro-partitions during a scan based on the filter predicate, to speed up queries |
| Creating a clustering key | `ALTER TABLE t CLUSTER BY (order_date);` — similar purpose to an index, boosts performance on frequently filtered columns |
| Who typically creates databases/schemas/roles | The Snowflake **admin** (ACCOUNTADMIN) |
| Which team performs unit testing of SQL scripts | The **developers** themselves, before code is promoted through CI/CD |
| ETL/ELT tools commonly paired with Snowflake | Azure Data Factory, Informatica, Talend, AWS Glue, Fivetran, Matillion, dbt, Airflow, etc. |

---

## 21. UDFs and UDTFs

**Concept:** User-Defined Functions (UDFs) let you build custom logic beyond built-in functions; User-Defined Table Functions (UDTFs) can return multiple rows/columns.

**Q1. What types of UDFs does Snowflake support?**
**A:** Scalar UDF (returns a single value per row), Tabular UDF/UDTF (returns a set of rows), and Secure UDF (hides the function's logic, similar in spirit to a secure view).

**Q2. How do you generate a dynamic "N-times multiplication table" using SQL (e.g., print the 5-times table, or make it work for any number passed in)?**
**A:** Combine a **recursive CTE** (to generate the sequence 1–10) with **string concatenation** to build each line, and wrap the logic in a **UDTF** so the multiplier becomes a parameter.
**Solution (concept walk-through):**
```sql
WITH RECURSIVE seq AS (
  SELECT 1 AS id
  UNION ALL
  SELECT id + 1 FROM seq WHERE id < 10
)
SELECT id, 5 || ' x ' || id || ' = ' || (5 * id) AS multiplication
FROM seq;
```
This static logic is then wrapped inside a `CREATE FUNCTION ... RETURNS TABLE (...) AS $$ ... $$` UDTF so the multiplier (5) becomes an input parameter.

---

## 22. Roles / Environments / SDLC (Process Questions)

**Concept:** Behavioral/process-style Snowflake developer interview questions about how teams typically work.

**Q1. What environments (databases/instances) does a typical Snowflake project have?**
**A:** Development (Dev), Testing, User Acceptance Testing (UAT)/SIT, Pre-Production/Sandbox, and Production. The first four are **non-production**; Production is the live environment.

**Q2. Who has full access to Production, and who typically deploys code there?**
**A:** Developers generally get full read/write access only in **Dev**; **Production** is largely read-only for developers. Code is promoted from Dev → Test/UAT → Production by the **DevOps team** using CI/CD pipelines (e.g., Jenkins), not manually — developers push their scripts to a code repository (GitHub/Bitbucket), and DevOps handles the actual deployment/migration.

**Q3. What's the typical daily workflow for a Snowflake developer team?**
**A:** Follow **Agile methodology** with daily stand-up/scrum calls; work is tracked as tickets/user stories (bug fixes, enhancements) in a tool like Jira, ServiceNow, Azure DevOps, or Monday.com; developers do **unit testing**, attach screenshots/results as sign-off evidence, then push code to the repository for the CI/CD pipeline to deploy.

---

## Quick-Fire "True/False" Set (Fail-safe & Time Travel)

These are commonly asked as rapid true/false checks:

| Statement | True/False |
|---|---|
| Only a Snowflake employee (support team) can recover data from Fail-safe | **True** |
| Fail-safe is a reliable way to create Dev/Test/other non-production environments | **False** (it's for production recovery only, via support request) |
| Fail-safe is not available for tables that have Time Travel | **False** (every permanent table gets both Time Travel *and* Fail-safe) |
| Data stored as part of Fail-safe counts toward storage cost billed to the customer | **True** |
| Fail-safe can be configured or disabled | **False** |

---

*End of compiled Q&A. Source: 29 transcribed Snowflake/SQL interview-preparation videos (compiled-transcripts.md).*
