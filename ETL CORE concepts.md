| #  | Topic from Your ETL CORE Concepts                            | ThinkETL Coverage                                                                                                  | Best ThinkETL Article / Resource                                                                                                                                                                       |
| -- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1  | **Lookup**                                                   | ✅ Direct                                                                                                           | [Lookup Transformation in Informatica Cloud (IICS)](https://thinketl.com/lookup-transformation-in-informatica-cloud-iics/)                                                                             |
| 2  | **Lookup vs Joiner**                                         | 🟡 Two articles — compare both                                                                                     | [Lookup Transformation](https://thinketl.com/lookup-transformation-in-informatica-cloud-iics/) + [Joiner Transformation](https://thinketl.com/joiner-transformation-in-informatica-cloud-iics/)        |
| 3  | **Connected vs Unconnected Lookup**                          | 🟡 Lookup article / catalogue coverage                                                                             | [Lookup Transformation](https://thinketl.com/lookup-transformation-in-informatica-cloud-iics/)                                                                                                         |
| 4  | **Cache Types — Static, Dynamic, Persistent, Shared**        | ✅ Strong for Lookup and Dynamic Lookup                                                                             | [Lookup Transformation](https://thinketl.com/lookup-transformation-in-informatica-cloud-iics/) + [Dynamic Lookup](https://thinketl.com/dynamic-lookup-in-informatica-cloud-iics/)                      |
| 5  | **Full Load vs Incremental Load**                            | 🟡 Related ETL implementation content; no single direct article verified                                           | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 6  | **CDC — Change Data Capture**                                | 🟡 No direct ThinkETL article verified specifically for generic CDC                                                | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 7  | **SCD Type 1**                                               | ✅ Direct                                                                                                           | [Implement SCD Type 1 Mapping in IICS](https://thinketl.com/implementing-scd-type-1-mapping-in-informatica-cloud-iics/)                                                                                |
| 8  | **SCD Type 2**                                               | ✅ Direct                                                                                                           | [Implement SCD Type 2 Mapping in IICS](https://thinketl.com/implementing-scd-type-2-mapping-in-informatica-cloud-iics/)                                                                                |
| 9  | **SCD Type 2 Using Dynamic Lookup**                          | ✅ Excellent                                                                                                        | [SCD Type 2 Using Dynamic Lookup](https://thinketl.com/scd-type-2-using-dynamic-lookup-in-informatica-iics/)                                                                                           |
| 10 | **SCD Type 3**                                               | ✅ Direct                                                                                                           | [Implement SCD Type 3 Mapping in IICS](https://thinketl.com/implementing-scd-type-3-mapping-in-informatica-cloud-iics/)                                                                                |
| 11 | **Staging Area / Landing Zone**                              | 🟡 Snowflake/data-loading related content; no single generic staging article verified                              | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 12 | **Surrogate Keys vs Natural Keys**                           | 🟡 Covered practically in SCD implementations                                                                      | [SCD Type 1](https://thinketl.com/implementing-scd-type-1-mapping-in-informatica-cloud-iics/) + [SCD Type 2 Dynamic Lookup](https://thinketl.com/scd-type-2-using-dynamic-lookup-in-informatica-iics/) |
| 13 | **Router vs Filter**                                         | 🟡 Router/Filter transformation coverage                                                                           | [IICS Transformations Guide](https://thinketl.com/informatica-cloud-iics-transformations-guide/)                                                                                                       |
| 14 | **Update Strategy / MERGE / Upsert**                         | 🟡 Practical implementation through SCD mappings                                                                   | [SCD Type 1 Mapping](https://thinketl.com/implementing-scd-type-1-mapping-in-informatica-cloud-iics/)                                                                                                  |
| 15 | **Aggregator**                                               | ✅ Direct                                                                                                           | [Aggregator Transformation in Informatica Cloud](https://thinketl.com/aggregator-transformation-in-informatica-cloud-iics/)                                                                            |
| 16 | **Sorter**                                                   | ✅ Direct                                                                                                           | [Sorter Transformation in Informatica Cloud](https://thinketl.com/sorter-transformation-in-informatica-cloud-iics/)                                                                                    |
| 17 | **Rank**                                                     | ✅ Direct                                                                                                           | [Rank Transformation in Informatica Cloud](https://thinketl.com/rank-transformation-in-informatica-cloud-iics/)                                                                                        |
| 18 | **Deduplication**                                            | ✅ Practical article                                                                                                | [Comparing Current Record with Previous Record / Duplicate Detection](https://thinketl.com/comparing-current-record-with-previous-record/)                                                             |
| 19 | **Error Handling — Rejected Rows / Bad Files / Dead Letter** | 🟡 No direct generic dead-letter article verified                                                                  | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 20 | **Reusable Transformations / Mapplets / Subjobs**            | ✅ Mapplet coverage                                                                                                 | [ThinkETL Catalogue – Mapplet Transformation](https://thinketl.com/catalogue/)                                                                                                                         |
| 21 | **Parameterization & Variables**                             | 🟡 ThinkETL catalogue/resource coverage; exact article can be selected from catalogue                              | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 22 | **Partitioning & Parallelism**                               | 🟡 No direct dedicated article verified                                                                            | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 23 | **Pushdown Optimization / ELT vs ETL**                       | 🟡 Informatica and Snowflake concepts available; no single exact match verified                                    | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 24 | **Watermark / High-Water Mark**                              | ❌ No direct ThinkETL article verified                                                                              | —                                                                                                                                                                                                      |
| 25 | **Idempotency**                                              | ❌ No direct ThinkETL article verified                                                                              | —                                                                                                                                                                                                      |
| 26 | **Fact & Dimension Load Order**                              | 🟡 SCD articles help with dimension loading, but no direct load-order article verified                             | [SCD Type 2 Using Dynamic Lookup](https://thinketl.com/scd-type-2-using-dynamic-lookup-in-informatica-iics/)                                                                                           |
| 27 | **Workflow / Job / Session**                                 | 🟡 IICS task/orchestration content; no single exact PowerCenter-equivalent article selected                        | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 28 | **Scheduling & Orchestration**                               | 🟡 No single direct generic article verified                                                                       | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |
| 29 | **Data Quality Checks**                                      | 🟡 Can be implemented using Expression, Filter, Router and Lookup                                                  | [IICS Transformations Guide](https://thinketl.com/informatica-cloud-iics-transformations-guide/)                                                                                                       |
| 30 | **Fivetran / No-Code ETL**                                   | ❌ ThinkETL is not the best resource for Fivetran-specific concepts                                                 | —                                                                                                                                                                                                      |
| 31 | **Performance Tuning**                                       | 🟡 Related optimization concepts available across transformations; catalogue is the safest verified starting point | [ThinkETL Catalogue](https://thinketl.com/catalogue/)                                                                                                                                                  |


# ETL Core Concepts — Tool-Agnostic Guide (Informatica / Talend / Fivetran)

> The idea: **tools change, concepts don't.** Once you understand *why* a Lookup exists or *why* CDC matters, you can pick up Informatica, Talend, dbt, Fivetran, ADF, or any new tool in a week — because you already know *what* problem it's solving. This guide explains each concept in plain language, why it exists, a real scenario, how it looks in each tool, and interview Q&A in the way you'd actually explain it to a colleague — not textbook definitions.

## Table of Contents

1. [Lookup — The Most Asked Concept](#1-lookup--the-most-asked-concept)
2. [Lookup vs Joiner (the classic interview trap)](#2-lookup-vs-joiner-the-classic-interview-trap)
3. [Connected vs Unconnected Lookup](#3-connected-vs-unconnected-lookup)
4. [Cache Types — Static, Dynamic, Persistent, Shared](#4-cache-types--static-dynamic-persistent-shared)
5. [Full Load vs Incremental Load](#5-full-load-vs-incremental-load)
6. [CDC — Change Data Capture](#6-cdc--change-data-capture)
7. [Slowly Changing Dimensions (SCD Type 1/2/3)](#7-slowly-changing-dimensions-scd-type-123)
8. [Staging Area / Landing Zone](#8-staging-area--landing-zone)
9. [Surrogate Keys vs Natural Keys](#9-surrogate-keys-vs-natural-keys)
10. [Router vs Filter](#10-router-vs-filter)
11. [Update Strategy / Merge / Upsert](#11-update-strategy--merge--upsert)
12. [Aggregator, Sorter, Rank](#12-aggregator-sorter-rank)
13. [Deduplication](#13-deduplication)
14. [Error Handling — Rejected Rows, Bad Files, Dead-Letter Pattern](#14-error-handling--rejected-rows-bad-files-dead-letter-pattern)
15. [Reusable Transformations / Mapplets / Subjobs](#15-reusable-transformations--mapplets--subjobs)
16. [Parameterization & Variables](#16-parameterization--variables)
17. [Partitioning & Parallelism](#17-partitioning--parallelism)
18. [Pushdown Optimization / ELT vs ETL](#18-pushdown-optimization--elt-vs-etl)
19. [Watermark / High-Water Mark](#19-watermark--high-water-mark)
20. [Idempotency](#20-idempotency)
21. [Fact & Dimension Load Order](#21-fact--dimension-load-order)
22. [Workflow / Job / Session — Same Idea, Different Names](#22-workflow--job--session--same-idea-different-names)
23. [Scheduling & Orchestration](#23-scheduling--orchestration)
24. [Data Quality Checks](#24-data-quality-checks)
25. [Fivetran — How These Concepts Show Up in a No-Code Tool](#25-fivetran--how-these-concepts-show-up-in-a-no-code-tool)
26. [Performance Tuning — Practical Checklist](#26-performance-tuning--practical-checklist)
27. [Rapid-Fire Interview Q&A Bank](#27-rapid-fire-interview-qa-bank)
28. [Scenario-Based Interview Questions](#28-scenario-based-interview-questions)

---

## 1. Lookup — The Most Asked Concept

### What it actually is
A Lookup is just: **"go check another table/reference data for a matching value, and bring back some columns from it."** That's it. Same idea as `VLOOKUP` in Excel, or a `JOIN` in SQL — but built as a reusable transformation step inside your pipeline instead of writing SQL each time.

### Why it exists (the real reason)
Your source data almost never has everything you need. A sales file has `customer_id = 4521`, but you need the customer's *name*, *region*, and *segment* to load into your fact table. Rather than joining giant tables in SQL every time, ETL tools give you a Lookup component so you can:
- Enrich a row with reference/master data (customer name, product category, currency rate)
- Check if a record **already exists** in the target (to decide insert vs update)
- Validate a code against a reference/master list (e.g., is this country code valid?)

### Real scenario
You're loading daily orders. Each order row has `product_id`. You do a Lookup against the `DIM_PRODUCT` table to pull `product_name`, `category`, `unit_price` — instead of the source system sending you all that repeated info every single day.

### Advantages of using a Lookup (why not just join in the source query?)
- **Reusability** — one Lookup object, used across many mappings/jobs, instead of rewriting the join logic everywhere.
- **Caching** — the tool can cache the lookup table in memory once, so it doesn't hit the database again for every single row (huge performance win for large volumes).
- **Decoupling** — your main pipeline stays clean; lookup logic (filters, default values, overrides) lives in one place.
- **Handles "no match" gracefully** — you control exactly what happens when there's no matching row (default value, reject, flag, etc.), which is clunky to express in a plain SQL join.
- **Works across heterogeneous sources** — you can look up against a table, a flat file, or even a different database than your main source — something a single SQL join usually can't do.

### How it looks across tools
| Tool | What it's called | How it works |
|---|---|---|
| Informatica PowerCenter/IICS | Lookup Transformation | Connected/Unconnected, with cache options |
| Talend | tMap (lookup input flow) or dedicated tFileInputDelimited/tDBInput joined in tMap | Lookup is a second input flow into `tMap`, joined on a key |
| Fivetran | N/A directly (Fivetran just replicates raw data) | Lookup-style enrichment is usually done *after* landing, in dbt/SQL models downstream |
| SQL/dbt world | Just a `JOIN` | Same concept, expressed as SQL |

---

## 2. Lookup vs Joiner (the classic interview trap)

People mix these up constantly. Here's the practical difference, no textbook fluff:

| | Lookup | Joiner |
|---|---|---|
| **Purpose** | Fetch a *few* columns from a reference/small table for enrichment or existence-check | Combine two full data streams/pipelines together |
| **Data volume fit** | Best for smallish reference/dimension tables that can be cached | Best for joining two large data streams (fact-to-fact, or two big extracts) |
| **Source types** | Can look up against database table, flat file, or even in-memory data | Typically joins two *pipeline flows* (source-to-source) |
| **Caching** | Has explicit caching (static/dynamic/persistent) | Usually builds an in-memory join based on the smaller "master" side, but no "cache reuse across runs" concept like Lookup has |
| **When "no match"** | Very flexible — default values, "new row" flag, reject, etc. | Controlled by join type: inner/left/right/full outer |

**Simple rule I actually use:** if I'm enriching a stream with a handful of columns from a smaller, mostly-static table → **Lookup**. If I'm combining two big data flows that are both central to my logic → **Joiner**.

---

## 3. Connected vs Unconnected Lookup

### Connected Lookup
Sits directly inline in your data flow — data flows *into* it and *out of* it, like any other transformation. Every row passes through it.

**Use when:** you need lookup output for *every* row, or need multiple output columns, or need to pass multiple input values.

### Unconnected Lookup
Not directly connected in the pipeline. Instead, it's *called* like a function — from an expression, only when needed, and only returns **one column**.

**Use when:**
- You only need the lookup *conditionally* (e.g., only look up currency conversion if `currency != 'USD'`) — saves performance by skipping unnecessary calls.
- You need to call the same lookup from multiple places/transformations in the same mapping.
- You only need a single return value.

### Real-world reasoning (why this distinction matters)
An Unconnected Lookup is basically a performance and reusability trick — "don't do this expensive lookup unless you actually need to for this row." That's why it's called conditionally via an `IIF`/expression rather than being wired into the main flow.

---

## 4. Cache Types — Static, Dynamic, Persistent, Shared

Caching is where Lookup performance either wins or dies. This is one of the most practically important — and most interview-tested — Lookup concepts.

### Static Cache (default)
The lookup table is read into memory **once** at the start, and never changes during the run — even if the underlying table changes mid-run. Fast, but "frozen in time."

**Use when:** the reference table won't change during your run (most cases — product master, country codes, etc.)

### Dynamic Cache
The cache updates itself **as you insert new rows** during the same run. So if row 500 in your source is actually the same "new customer" that row 10 already inserted, the dynamic cache "knows" about it because it tracks inserts happening within the same session.

**Use when:** you're loading a table and simultaneously need to detect duplicates *within the same load* — classic example: loading a dimension table where new customers can appear multiple times in the same source file, and you don't want duplicate inserts.

### Persistent Cache
The cache is **saved to disk** after the run, so the *next* run can reuse it instead of rebuilding from scratch.

**Use when:** the reference table is huge and rarely changes (e.g., a massive product catalog refreshed only weekly) — rebuilding the cache every single run wastes time.

### Shared Cache
Multiple Lookup transformations (in the same or different mappings) reuse **the same cache** instead of each building its own copy.

**Use when:** several mappings all look up the same reference table (e.g., `DIM_DATE`, `DIM_CURRENCY`) — no point building 5 separate copies of the same cache in memory.

### Quick comparison table

| Cache Type | Rebuilt every run? | Updates during run? | Shared across lookups? | Typical use case |
|---|---|---|---|---|
| Static | Yes | No | No | Standard reference data |
| Dynamic | Yes | Yes (as you insert) | No | Dedup + insert/update decision in one pass |
| Persistent | No (reused) | No | No | Large, rarely-changing tables |
| Shared | Yes (once) | Depends | Yes | Same lookup table used in multiple places |

---

## 5. Full Load vs Incremental Load

### Full Load
Truncate (or drop/recreate) the target and reload **everything** from source, every time.

**When it's fine:** small tables, dimension tables that rebuild fast, one-time historical loads, or when source has no reliable "changed since" indicator.

**Downside:** doesn't scale — a 500-million-row fact table can't realistically be fully reloaded every night.

### Incremental Load
Only pull/process the rows that are **new or changed** since the last run.

**How you typically detect "changed":**
- A `last_updated_date` / `modified_date` column in the source (most common, simplest)
- An auto-increment ID or sequence you track (`load only rows with id > last_max_id`)
- Database transaction logs (true CDC — see next section)
- A hash comparison of the row (if no timestamp exists at all)

**Real scenario:** loading a `ORDERS` table with 50M rows. You don't reload all 50M every night — you only pull orders where `updated_at > last_successful_run_time`, maybe a few thousand rows a day.

### Interview-ready one-liner
> "Full load = simple but doesn't scale. Incremental load = efficient but needs a reliable way to detect what changed — that reliability is the hard part, not the loading itself."

---

## 6. CDC — Change Data Capture

### What it really means
CDC is a **specific technique for incremental loading** that captures inserts/updates/deletes as they happen, usually by reading the database's own transaction log — instead of querying the table itself.

### Why it's better than timestamp-based incremental loads
- **Detects deletes** — a `WHERE updated_at > X` query can never tell you a row was *deleted*. Log-based CDC can.
- **Lower load on the source system** — you're reading the transaction log, not running a heavy query against the live table.
- **Near real-time** — logs are typically streamed continuously (e.g., via Debezium, GoldenGate, Fivetran's log-based connectors), so downstream data is minutes or seconds behind, not hours.
- **Catches every change, even fast successive ones** — a timestamp-based approach can miss a row that was updated twice within the same polling window (you'd only see the latest state, not that it changed twice) — CDC via logs sees each individual change event.

### Types of CDC (in practice)

| Type | How it works | Example |
|---|---|---|
| Timestamp-based | Query `WHERE last_modified > last_run` | Simple, most common in batch ETL |
| Trigger-based | DB triggers write changes to a separate "shadow" table | Older-style, adds overhead to source DB |
| Log-based | Reads the database transaction log (WAL/redo log/binlog) | Debezium, Oracle GoldenGate, Fivetran's log-based connectors — gold standard for real-time CDC |
| Hash/checksum-based | Compare a hash of the current row to a stored hash from last run | Used when there's no timestamp or log access at all |

### How it shows up per tool
- **Informatica**: PowerExchange CDC, or IICS's CDC connectors reading DB logs.
- **Talend**: tCDC components pull from database change logs (log-based, works with Oracle/MySQL/SQL Server CDC features).
- **Fivetran**: this is literally Fivetran's core selling point — most database connectors use log-based CDC out of the box, so you barely configure anything; it's automatic.

---

## 7. Slowly Changing Dimensions (SCD Type 1/2/3)

This is a data-warehousing concept, not strictly an "ETL tool" concept — but every ETL tool has built-in support for it because it comes up constantly.

### The core problem
A customer's address changes. Do you:
(a) just overwrite the old address (lose history)?
(b) keep the old row and add a new one (keep full history)?
(c) keep just the *previous* value in a separate column (partial history)?

That decision is what SCD types are about.

### SCD Type 1 — Overwrite
Just update the existing row. No history kept.

**Use when:** you don't care about history — e.g., correcting a typo in a customer's name. You don't need to know it used to be spelled wrong.

### SCD Type 2 — Add a New Row (most common in real projects)
Keep the old row (mark it inactive/expired), insert a new row with the updated value, and typically track it with:
- `effective_start_date`, `effective_end_date`
- `is_current` flag (`Y`/`N`)
- Sometimes a `version_number`

**Use when:** you need full historical tracking — e.g., "which sales region was this customer in when they placed each historical order?" This is the standard for most dimension tables in real data warehouses.

**How ETL tools implement it:** Lookup against the dimension (using the natural/business key) → compare incoming values to the current row's values → if different, expire the old row (update `end_date`/`is_current`) and insert a new row with a new surrogate key.

### SCD Type 3 — Keep Previous Value in a Column
Add a `previous_value` column alongside the `current_value` column. Only tracks **one** prior state, not full history.

**Use when:** you only care about "what changed most recently" — rare in practice compared to Type 1 and 2, but occasionally used for things like "previous sales manager" vs "current sales manager."

### Quick comparison

| Type | History kept | Row count grows? | Complexity | Real-world usage |
|---|---|---|---|---|
| Type 1 | None | No | Low | Very common for non-critical attributes |
| Type 2 | Full | Yes | Medium-High | Most common in real DW projects |
| Type 3 | Only last value | No | Low | Rare, special cases only |

---

## 8. Staging Area / Landing Zone

### What it is
A temporary holding table/area where raw source data lands **before** any transformation or business logic is applied.

### Why it exists (the real reasons, not textbook ones)
- **Isolates source system impact** — you extract from the source once, quickly, and do all your heavy transformation logic against the staging copy instead of hammering the production source system repeatedly.
- **Restartability** — if your pipeline fails halfway through transformation, you don't need to re-extract from source (which might be slow, rate-limited, or already moved on) — you just re-run transformations against what's already staged.
- **Auditability** — you can always compare "what we received" (staging) vs "what we loaded" (target) to debug data issues.
- **Decouples extraction timing from transformation timing** — e.g., extract every 15 minutes into staging, but only run the heavier transformation/load job once an hour.

### Real scenario
Fivetran lands raw data into a `raw` schema (this *is* the staging/landing zone) with zero transformation — then dbt or your BI tool builds cleaned, business-ready models on top of it. Informatica/Talend jobs often do the same: dump to a staging table first, then a second job applies business rules and loads the final target.

---

## 9. Surrogate Keys vs Natural Keys

### Natural Key
A key that already exists in the source data and has business meaning — e.g., `customer_email`, `SSN`, `product_SKU`.

### Surrogate Key
An artificial key generated purely for warehouse purposes — usually a simple auto-incrementing integer — with **no business meaning at all**.

### Why surrogate keys matter (the real reason, beyond "it's a best practice")
- **Natural keys change.** A product's SKU might get reassigned, a customer's email might change. If your fact table joins on the natural key and that key changes, your historical joins silently break.
- **SCD Type 2 literally requires surrogate keys** — the same customer needs *multiple* rows (each representing a different time period), which is impossible if the primary key is the natural key alone.
- **Performance** — joining on a simple integer surrogate key is faster than joining on a long string/composite natural key across millions of fact rows.
- **Consistency across source systems** — if you're merging customer data from 3 different source systems, each with their own ID scheme, a surrogate key gives you one unified key regardless of source.

---

## 10. Router vs Filter

Both are used to split/filter data based on conditions — but differently.

### Filter
One input, **one** condition, rows either pass or get dropped. Binary — no multiple output paths.

### Router
One input, **multiple** conditions ("groups"), each with its own output path — plus a default/"unmatched" group. Like a `CASE WHEN` that sends rows to entirely different downstream paths, not just filters them out.

**Real scenario:** You have one incoming orders stream. You want:
- Orders > $10,000 → go to a "high value" table for special handling
- Orders from EU countries → go to a GDPR-compliant table
- Everything else → standard table

A **Router** does this in one pass (multiple output groups). A **Filter** would require 3 separate Filter transformations, each re-scanning the same data — much less efficient.

---

## 11. Update Strategy / Merge / Upsert

### What it is
The logic that decides, **row by row**, whether an incoming record should be an **INSERT**, **UPDATE**, **DELETE**, or **REJECT** in the target.

### Why it's needed
Without this, every load would either fully overwrite the target (full load) or blindly insert (creating duplicates). Real incremental loads need row-level decisions: "this customer already exists → update their address; this customer is new → insert."

### How it's typically implemented
1. Lookup the target table using the business/natural key.
2. If no match found → mark as **INSERT**.
3. If match found and any tracked column differs → mark as **UPDATE**.
4. If match found and nothing differs → **do nothing** (skip — don't waste a write).
5. (Optional, less common) if source signals a delete → mark as **DELETE**.

### Modern equivalent
This is exactly what a SQL `MERGE` statement (or `INSERT ... ON CONFLICT DO UPDATE` in Postgres, or dbt's incremental `merge` strategy) does — same logic, just expressed as SQL instead of a GUI transformation.

| Tool | How it's expressed |
|---|---|
| Informatica | Update Strategy transformation (`DD_INSERT`, `DD_UPDATE`, `DD_DELETE`, `DD_REJECT`) |
| Talend | tMap's "Insert/Update/Delete" row settings, or a dedicated `MERGE` in tDBOutput |
| Fivetran | Handled automatically — Fivetran does upserts under the hood based on the source's primary key |
| SQL/dbt | `MERGE INTO target USING source ON (key) WHEN MATCHED ... WHEN NOT MATCHED ...` |

---

## 12. Aggregator, Sorter, Rank

Grouped together because they're often confused or used together in the same mapping.

### Aggregator
Groups rows and computes summary values — `SUM`, `COUNT`, `AVG`, `MAX`, `MIN` per group. Same idea as SQL's `GROUP BY`.

**Real scenario:** total daily sales per store — group by `store_id`, `date`, sum `sales_amount`.

### Sorter
Orders rows by one or more columns. Sounds trivial, but matters a lot because:
- Some downstream transformations (like Aggregator, Joiner in "sorted input" mode) run **much faster** if the input is already sorted — the tool doesn't have to buffer/sort in memory itself.
- Needed before rank/dedup logic that depends on order (e.g., "keep only the most recent row per customer" needs a sort by date first).

### Rank
Picks the **top-N** (or bottom-N) rows within each group — e.g., "top 3 highest-paid employees per department." Similar to SQL's `ROW_NUMBER()`/`RANK()` window functions.

**Real scenario:** "get the most recent transaction per account" → Rank by `transaction_date DESC`, partition by `account_id`, keep rank = 1.

---

## 13. Deduplication

### Why duplicates happen (in real pipelines, not theory)
- Source system sends the same file twice (retry after a network blip)
- CDC captures multiple changes to the same row within one batch window
- A join accidentally produces a fan-out (one row matching multiple rows on the other side)
- Manual re-runs of a failed job without proper cleanup first

### Common dedup techniques
1. **Aggregator with GROUP BY on all columns** — crude but works for exact duplicate rows.
2. **Rank/Window function** — `ROW_NUMBER() OVER (PARTITION BY key ORDER BY updated_at DESC) = 1` — the standard, most-used technique. Keeps only the "latest" or "best" row per key.
3. **Dynamic Lookup cache** — as discussed in section 4, useful for catching duplicates *within the same load* before they're inserted.
4. **Distinct** — simplest, but only works for true exact-row duplicates, and can be expensive on huge datasets.

### Interview-ready one-liner
> "Almost every real dedup problem in ETL boils down to: partition by the business key, order by recency, keep row #1. Everything else is a variation of that."

---

## 14. Error Handling — Rejected Rows, Bad Files, Dead-Letter Pattern

### Why this matters more than people think
A pipeline that silently drops bad rows is a pipeline nobody trusts. Good ETL design always answers: *"if this row fails, where does it go, and how do we know it happened?"*

### Common patterns
- **Reject/error tables** — bad rows get written to a separate `_ERROR` or `_REJECTED` table instead of vanishing, along with the reason for rejection.
- **Row-level error handling in Update Strategy** — e.g., `DD_REJECT` in Informatica routes rows that fail business validation to a reject file automatically.
- **`ON_ERROR = CONTINUE`-style behavior** (seen in bulk loaders/Snowflake, Talend's "Die on error" toggle, etc.) — skip the bad row, keep processing the good ones, log what got skipped.
- **Dead-letter queue pattern** (more common in streaming/real-time ETL, e.g., Kafka-based pipelines) — failed messages go to a separate topic/queue for manual review instead of blocking the whole stream.
- **Audit/control tables** — every run logs: rows read, rows loaded, rows rejected, error reason, timestamp — so failures are visible without digging through logs.

### Real scenario
Loading customer records where `email` is a mandatory field. Rows with a null email get routed to `CUSTOMER_ERROR` table with a reason code (`MISSING_EMAIL`), while all valid rows continue loading normally — instead of the whole job failing over one bad row.

---

## 15. Reusable Transformations / Mapplets / Subjobs

### The core idea
If you find yourself building the exact same logic (e.g., "standardize phone number format," "look up currency conversion") in multiple pipelines, package it **once** and reuse it everywhere — instead of copy-pasting and maintaining N copies.

### Per tool
| Tool | What it's called |
|---|---|
| Informatica | Mapplet (reusable group of transformations) / Reusable Transformation (single transformation) |
| Talend | Subjob (a group of components reused within a job) / Joblet (a reusable mini-job called from multiple jobs) |
| Fivetran | N/A — Fivetran itself doesn't have custom transformation logic; reusability lives in your dbt models downstream |
| SQL/dbt | A macro, or a shared model/CTE referenced by multiple downstream models |

### Why this matters for interviews
It signals you think about **maintainability**, not just "does it work once." If a business rule changes (e.g., how you calculate "customer lifetime value"), you want to change it in **one place**, not hunt through 15 pipelines.

---

## 16. Parameterization & Variables

### What it is
Instead of hardcoding values (file paths, dates, table names, thresholds) directly into your pipeline, you use **parameters/variables** that get set at runtime.

### Why it's essential in real projects
- **Same pipeline, different environments** — dev/QA/prod use different connection strings, schemas, file paths. Parameters let you deploy the same logic everywhere without editing the pipeline itself.
- **Reusability** — one generic pipeline (e.g., "load any table from S3 folder X into Snowflake table Y") driven entirely by parameters, instead of building a new pipeline per table (this is literally the pattern used in the "dynamic multi-folder ingestion" Snowflake scenario — same concept, ETL-tool version).
- **Incremental load control** — a variable tracking "last successful run timestamp" is what makes incremental loads possible without hardcoding dates.

### Types (conceptually the same across tools)
- **Session/job parameters** — set once per run (e.g., which environment, which date range).
- **Mapping/pipeline variables** — can change value *during* the run (e.g., a running total, or "max date seen so far" which then gets persisted for next run's incremental load).
- **System/global parameters** — shared across many jobs (e.g., a shared connection string, a shared "run_id").

---

## 17. Partitioning & Parallelism

### What it is
Splitting a large data load into multiple chunks that process **simultaneously**, instead of one row/file at a time sequentially.

### Why it matters
A 100-million-row load that takes 4 hours sequentially might take 45 minutes if split into 8 parallel partitions (assuming your source/target and hardware can handle the concurrent load).

### Common partitioning strategies
- **Key-range partitioning** — split by a value range (e.g., `customer_id 1–1M`, `1M–2M`, etc.)
- **Hash partitioning** — split rows evenly based on a hash of a key column, avoiding "hot" partitions if data isn't evenly distributed by range.
- **Round-robin** — rows distributed evenly regardless of value, simplest but doesn't help if you need related rows to land in the same partition (e.g., for aggregation).
- **Pass-through** — inherits partitioning already set by an earlier step, avoiding an unnecessary re-shuffle.

### Real-world caveat (worth mentioning in interviews)
Partitioning helps most when **both source and target can handle concurrent connections/writes** without becoming the bottleneck themselves — over-partitioning against a source database that can't handle 20 concurrent connections just creates contention, not speed.

---

## 18. Pushdown Optimization / ELT vs ETL

### ETL vs ELT — the real difference
- **ETL (Extract, Transform, Load)** — transform the data *before* loading into the target, usually in the ETL tool's own engine.
- **ELT (Extract, Load, Transform)** — load raw data into the target first, then transform it **using the target's own compute** (e.g., Snowflake/BigQuery SQL, dbt models).

### Why ELT has become more popular
Modern cloud warehouses (Snowflake, BigQuery, Redshift) are extremely fast at large-scale SQL transformations — often faster than an ETL tool's own engine. So instead of pulling data out, transforming it in a separate engine, and pushing it back, you let the warehouse do the heavy lifting.

### Pushdown Optimization
This is the *hybrid* concept — even a traditional ETL tool (Informatica, Talend) can be told: "instead of processing this transformation logic yourself, generate SQL and push it down to run inside the source or target database." This gets you ETL-tool convenience with ELT-style performance.

### Where Fivetran fits
Fivetran is fundamentally an **EL** tool (Extract + Load only) — it deliberately does **not** transform data. The "T" is left entirely to downstream tools like dbt, which is itself a pure ELT/transformation tool running SQL directly in the warehouse. This is a deliberate design philosophy, not a missing feature.

---

## 19. Watermark / High-Water Mark

### What it is
The **marker** (usually a timestamp or an ID) that tells your incremental load "everything up to this point has already been processed — next run, start from here."

### Why it's more subtle than it sounds
- If you use `CURRENT_TIMESTAMP()` as your watermark at the *start* of a run, but the run takes 20 minutes, you might miss rows that were inserted into the source *during* those 20 minutes (their timestamp falls before your watermark but they weren't there yet when you queried).
- Real implementations often use a **safety buffer** ("look back 10 minutes earlier than the strict last watermark") to avoid missing late-arriving or slightly-out-of-order data.
- The watermark itself needs to be **persisted somewhere durable** (a control table, a file, a parameter store) — not just an in-memory variable that disappears if the job crashes mid-run.

### Real scenario
A daily incremental job stores `last_loaded_timestamp = 2026-08-24 23:58:00` in a control table after a successful run. Tomorrow's run reads that value, pulls everything `WHERE updated_at > '2026-08-24 23:58:00'`, and updates the watermark again on success — but only **after** confirming the load succeeded, so a failed run doesn't advance the watermark and silently skip data.

---

## 20. Idempotency

### What it means in plain terms
**Running the same job twice should not produce a different (or duplicated) result than running it once.**

### Why it's one of the most important — and most overlooked — ETL concepts
Jobs fail. Networks blip. Someone accidentally re-triggers a pipeline. If your job isn't idempotent, a simple re-run after a failure can silently double-insert data, corrupt totals, or create duplicate dimension rows.

### How you make a load idempotent, in practice
- Use **upserts/merge** (see section 11) instead of blind inserts — re-running just re-updates the same rows instead of duplicating them.
- **Delete-then-insert by partition/date** — e.g., before loading "yesterday's data," delete any existing rows for yesterday first, then insert fresh — safe to re-run any number of times.
- Track **processed file/batch IDs** in a control table (like the `FILE_LOAD_LOG` pattern from earlier Snowflake examples) so a file already processed is automatically skipped on a re-run.
- Avoid relying on "append-only, always insert" logic for anything that might realistically get re-run.

---

## 21. Fact & Dimension Load Order

### Why order matters
A fact table row typically references dimension surrogate keys (e.g., `customer_key`, `product_key`, `date_key`). If you load the fact table **before** the dimensions are updated, you either get failed foreign key lookups, or — worse — the fact row silently gets a "default/unknown" key and you lose the real linkage.

### The practical rule
**Always load dimensions before facts.** In real pipelines this usually means:
1. Load/update all relevant dimension tables first (customer, product, date, etc.)
2. Then run the fact load, which does Lookups against the now-updated dimensions to fetch the correct surrogate keys.

### Common real-world exception
Some pipelines use a **"late-arriving dimension" / "inferred member"** pattern: if a fact row references a customer that doesn't exist in the dimension *yet*, insert a placeholder dimension row (with just the natural key, other attributes as "unknown") so the fact load doesn't fail — then backfill the dimension attributes properly when the real customer data arrives later.

---

## 22. Workflow / Job / Session — Same Idea, Different Names

This trips people up switching tools, so here's the direct mapping:

| Concept | Informatica | Talend | Fivetran | Generic term |
|---|---|---|---|---|
| A single transformation flow (source → transform → target) | Mapping | Job (a single `.item` design) | Connector Sync | Pipeline |
| A runnable unit that executes a mapping with runtime settings | Session | Job execution/context | Sync schedule/run | Task/Run |
| A group of sessions/tasks with control logic (order, conditions) | Workflow | Job with subjobs, or orchestrated via Talend's Job orchestration | N/A (Fivetran doesn't orchestrate multi-step logic — that's dbt's/Airflow's job) | Orchestration/DAG |
| A reusable chunk of logic | Mapplet / Reusable Transformation | Joblet / Subjob | N/A | Module/macro |

**Interview-ready framing:** *"Different tools use different words — mapping vs job, session vs run, workflow vs orchestration — but they all map to the same three layers: the transformation logic itself, the runtime execution of that logic, and the higher-level scheduling/dependency control around multiple runs."*

---

## 23. Scheduling & Orchestration

### The difference (often confused)
- **Scheduling** = "run this at 2 AM every day." Time-based trigger.
- **Orchestration** = "run job A, then only if it succeeds run job B and C in parallel, then run job D once both finish." Dependency-based control across multiple jobs.

### Why orchestration matters more as pipelines grow
A single job's internal schedule is easy. The real complexity comes when you have 30 interdependent pipelines (extract → stage → dimension loads → fact loads → aggregate/report layer) and need to guarantee correct order, handle partial failures, and avoid re-running things unnecessarily.

### Common tools/approaches
- Native tool schedulers (Informatica Workflow Manager scheduling, Talend's TAC scheduler)
- External orchestrators — **Airflow**, **Control-M**, **Azure Data Factory pipelines**, **dbt Cloud jobs** — increasingly common because they can coordinate across *multiple different tools* (e.g., "wait for Fivetran sync to finish, then trigger dbt, then trigger a Talend job"), which a single tool's own scheduler usually can't do.

---

## 24. Data Quality Checks

### Where DQ checks typically live in a pipeline
1. **At source/staging** — row count checks, null checks on mandatory fields, data type/format validation, duplicate checks.
2. **Mid-pipeline** — business rule validation (e.g., "order amount can't be negative," "end date must be after start date").
3. **Post-load** — reconciliation checks (source row count vs target row count), referential integrity checks (every fact row has a valid dimension key).

### Common practical checks
- **Row count reconciliation** — did we load the same number of rows we extracted (minus expected rejects)?
- **Null/completeness checks** — are mandatory fields actually populated?
- **Uniqueness checks** — is the primary/business key actually unique after load?
- **Referential integrity** — do all foreign keys in the fact table exist in the dimension tables?
- **Freshness checks** — did today's load actually run, and is the data recent (not stale from 3 days ago due to a silent failure)?

### Real-world framing for interviews
> "Good ETL isn't just 'did the job run without erroring out' — it's 'did the job produce data we can actually trust,' which is a completely different, and honestly harder, question."

---

## 25. Fivetran — How These Concepts Show Up in a No-Code Tool

Since Fivetran deliberately hides most "transformation" concepts (it's EL, not ETL), it's worth being explicit about what maps where, since interviewers sometimes test whether you understand *why* Fivetran looks so different.

| Concept | How Fivetran handles it |
|---|---|
| Lookup / enrichment | Not done in Fivetran — happens downstream in dbt/SQL after raw data lands |
| CDC | Built-in and automatic for most database connectors (log-based) — you don't configure Lookup/cache logic yourself |
| Incremental load | Automatic — Fivetran tracks state per connector/table internally |
| SCD Type 2 | Fivetran has a "History Mode" feature for some connectors that automatically keeps a full change history — conceptually the same idea as SCD Type 2, but automated |
| Staging | The "raw" schema Fivetran lands data into effectively *is* the staging layer |
| Error handling | Handled at the connector level — failed syncs alert you, but there's no per-row "reject table" concept the way Informatica/Talend expose |
| Reusable logic | N/A within Fivetran itself — all business logic reusability happens in dbt models downstream |
| Idempotency | Built-in — re-running/re-syncing a connector doesn't duplicate data, since it's upsert-based under the hood |

**Why this matters for interviews:** if asked "how would you do a Lookup in Fivetran," the honest, correct answer is *"you wouldn't — Fivetran only extracts and loads; that enrichment logic would live in a dbt model or SQL view built on top of the raw Fivetran tables."* Saying this shows you actually understand the tool's philosophy, not just its UI.

---

## 26. Performance Tuning — Practical Checklist

Concepts, not tool-specific screenshots — applies everywhere:

- **Filter early, filter at the source** — don't pull 10 million rows into your pipeline just to filter down to 10,000 later; push the filter into the source query/staging step if possible.
- **Cache smartly** — use persistent/shared cache for reference data that doesn't change often (see section 4).
- **Avoid unnecessary sorts** — sorting is expensive; only sort when a downstream step actually requires it.
- **Push down what you can** — let the source/target database do heavy joins/aggregations if it's faster there than in the ETL engine (see section 18).
- **Partition large loads** — but don't over-partition against a source that can't handle the concurrency (see section 17).
- **Incremental over full wherever possible** — the single biggest performance win for large tables.
- **Batch commits, don't commit row-by-row** — committing every single row to the target is dramatically slower than batching (e.g., commit every 10,000 rows).
- **Avoid unnecessary Lookups per row** — if a value rarely changes, consider whether you really need a live lookup vs. a cached/pre-joined reference set.
- **Monitor and log run times per stage**, not just overall job time — so you actually know *which* step is the bottleneck instead of guessing.

---

## 27. Rapid-Fire Interview Q&A Bank

**Q1. What's a Lookup transformation, in your own words?**
It's how you pull extra columns from another table/reference data to enrich your main data flow — same as a SQL join, just built as a reusable step with caching built in.

**Q2. When would you use an unconnected Lookup over a connected one?**
When I only need the lookup conditionally, not for every row, or I want to call the same lookup from multiple places in the mapping — it behaves like a function you call on demand rather than something every row has to pass through.

**Q3. What's the difference between static and dynamic cache?**
Static cache is built once and frozen for the run. Dynamic cache updates itself as you insert new rows during the same run — so it can catch duplicates that appear *within* the same load, not just against what was already in the table before the run started.

**Q4. Why would you use incremental load instead of full load?**
Performance and scale, mainly. Reloading a 500-million-row table every night doesn't make sense when maybe 10,000 rows actually changed. Incremental load only touches what changed.

**Q5. What's the risk with timestamp-based incremental loads?**
They can't detect deletes, and if a row gets updated twice quickly within the same polling window, you might only capture its final state, not that it changed twice. That's where log-based CDC does better.

**Q6. Explain SCD Type 2 like I'm not a data engineer.**
Instead of overwriting a customer's old address, you keep the old row, mark it as no-longer-current, and add a new row with the new address. That way, historical reports still show the correct address at the time each order happened, not today's address applied backward.

**Q7. Why do we need surrogate keys if the source already has a primary key?**
Because natural keys can change, get reused, or differ across source systems. Surrogate keys give you a stable, warehouse-only identifier — and they're required for SCD Type 2, since the same customer needs multiple rows over time, which isn't possible if the primary key is the natural key alone.

**Q8. What's the difference between Router and Filter?**
Filter has one condition, rows either pass or get dropped. Router has multiple conditions, each with its own output path, evaluated in a single pass — so you're not scanning the same data multiple times like you would with several Filters chained together.

**Q9. How does an Update Strategy / Merge actually decide insert vs update?**
Look up the target using the business key. No match → insert. Match found but values differ → update. Match found and nothing changed → skip, don't waste a write.

**Q10. Why is idempotency important in ETL?**
Because jobs fail and get re-run. If a re-run causes duplicate inserts or double-counted totals, you can't trust re-running a failed job — and in production, failures *will* happen, so this isn't optional.

**Q11. What's a watermark, and what's the tricky part about it?**
It's the marker of "how far we've already loaded." The tricky part is timing — if you grab the watermark at the *start* of a long-running job, you might miss rows inserted into the source while your job was still running. A small safety buffer usually fixes that.

**Q12. Why load dimensions before facts?**
Because fact rows reference dimension surrogate keys. If the dimension isn't updated yet, the fact load either fails the lookup or falls back to an "unknown" key, losing the real relationship.

**Q13. What's the difference between ETL and ELT?**
ETL transforms data before loading it into the target, usually inside the ETL tool's own engine. ELT loads raw data first and transforms it afterward using the target system's compute (e.g., SQL in Snowflake, or dbt). ELT became popular because modern cloud warehouses are often faster at large-scale transforms than a separate ETL engine.

**Q14. If Fivetran doesn't do transformations, what does it actually do?**
Extract and Load only — it replicates source data as-is (usually via log-based CDC) into a raw schema in your warehouse. All transformation logic (joins, business rules, SCD, aggregations) is expected to happen afterward, typically in dbt.

**Q15. How do you handle bad/rejected records in a pipeline?**
Route them to a separate error/reject table with a reason code instead of letting them silently fail or blocking the whole load — and log counts so you know how many rows were rejected and why, without digging through raw logs.

**Q16. What's the point of a staging area if you're just going to transform the data anyway?**
It isolates the source system from your transformation logic, lets you restart mid-pipeline without re-extracting, and gives you an audit trail — "what did we actually receive" vs "what did we load."

**Q17. What's pushdown optimization?**
Telling the ETL tool to generate SQL and let the source/target database execute the transformation logic itself, instead of pulling data into the ETL engine to process it — usually faster for large volumes.

**Q18. How would you dedupe records in almost any ETL tool?**
Partition by the business key, order by whatever makes a row "the best" version (usually most recent), keep only rank/row-number 1. Same logic whether you express it as a Rank transformation, a window function in SQL, or a dynamic lookup cache check.

**Q19. What's the difference between a mapping-level parameter and a session/job-level parameter?**
A session/job parameter is set once at the start of the run (like an environment or a date range). A mapping/pipeline variable can actually change value *during* the run — e.g., tracking the max date seen so far, which then gets persisted for the next incremental run.

**Q20. Why might partitioning a load actually make it slower, not faster?**
If the source or target can't handle that many concurrent connections/writes, you create contention instead of parallel speed — partitioning only helps if both ends of the pipeline can genuinely handle the concurrency.

---

## 28. Scenario-Based Interview Questions

**Scenario 1: "We're loading a 200-million-row fact table nightly, and it's taking 6 hours. How would you speed it up?"**
Walk through it step by step, not just one answer:
- First check: is this a full load or incremental? If full, switch to incremental — this alone is usually the biggest win.
- Check partitioning — is the load parallelized, and can source/target actually support more concurrency?
- Check for unnecessary sorts or row-by-row lookups that could be cached or pushed down.
- Check commit frequency — row-by-row commits vs. batched commits make a huge difference.
- Consider pushdown — could some of this transformation run as SQL directly in the warehouse instead of the ETL engine?

**Scenario 2: "A dimension table is missing 200 rows that should have been there since yesterday's load. How do you debug it?"**
- Check the control/audit log for yesterday's run — did it actually complete successfully, or fail partway?
- Check the watermark — did it advance correctly, or did a failed run leave it stuck, causing today's incremental load to also skip the same window?
- Check the reject/error table — were those 200 rows rejected due to a data quality issue (e.g., a null business key)?
- Check if it's a late-arriving dimension issue — did the fact table reference them before the dimension load ran, generating a placeholder row instead?

**Scenario 3: "The business wants to know 'what a customer's address was at the time of each historical order,' but we're currently overwriting the address (Type 1). What do you do?"**
Explain that you'd migrate the dimension to SCD Type 2 — add `effective_start_date`, `effective_end_date`, `is_current` — and note the tricky part: historical fact rows already loaded were joined against the *old*, Type-1-only version of the dimension, so a full historical reload of the fact table (or at least a key re-mapping) is usually needed to actually get correct historical answers going forward — just switching the dimension logic alone doesn't fix already-loaded fact data.

**Scenario 4: "A job that used to run in 10 minutes now takes 2 hours, and nothing in the code changed. What do you check?"**
- Data volume — has source volume genuinely grown (organic growth, or a bad actor/system suddenly sending 100x normal volume)?
- Source-side issues — is a lookup table that used to be small now much bigger, breaking a static cache assumption?
- Infrastructure — is a shared resource (network, database, ETL server) under more load from other jobs?
- Statistics/indexes — if pushdown SQL is involved, are the source/target table statistics stale, causing the database's own query optimizer to pick a bad plan?

**Scenario 5: "How would you migrate an existing Informatica pipeline's logic to Fivetran + dbt?"**
- Anything that's pure extract/load (pulling data as-is from a source database or SaaS API) → replace with a Fivetran connector; CDC and incremental logic become automatic instead of manually configured.
- Anything that's transformation logic (lookups, SCD, aggregations, business rules) → move into dbt models, written as SQL, running against the raw Fivetran-landed tables.
- Reusable Informatica mapplets → become dbt macros or shared models.
- Orchestration/scheduling that previously lived in Informatica Workflow Manager → moves to dbt Cloud jobs or an external orchestrator (Airflow), which can also wait for the Fivetran sync to finish before triggering the dbt run.

---

## Closing Note

If you remember nothing else from this doc, remember this framing for interviews:

> "Every ETL tool is really just giving you a UI/API around the same handful of ideas: get data (extract), decide what changed (CDC/incremental logic), enrich and reshape it (lookup, joins, aggregations), decide insert/update/delete (merge logic), track history if needed (SCD), and load it reliably and repeatably (idempotency, error handling, staging). Learn those concepts once, and any tool — old or new — is just new syntax around the same problems."
