# PostgreSQL Commands & Use Cases (Render.com)

A practical reference for inspecting and maintaining a Render-hosted PostgreSQL database using the `psql` terminal.

> **Public repository note:** This document intentionally uses placeholder database names, usernames, and hostnames. Never commit real database credentials, connection strings, passwords, API keys, or other secrets to a public repository.

> **Scope:** PostgreSQL administration, monitoring, troubleshooting, storage, connections, and routine maintenance.
>
> **Important:** Most inspection queries are safe to run at any time. Commands such as `VACUUM FULL`, `REINDEX`, `DROP`, `TRUNCATE`, and some `ALTER TABLE` operations can have significant production impact and should be used carefully.

---

## 1. 🔑 PostgreSQL Version Information

### Client Version

Run these commands **outside** the `psql` prompt:

    psql --version

or:

    psql -V

The client version tells you which `psql` program is installed on your local machine.

### Server Version

Run inside `psql`:

    SELECT version();

A shorter version:

    SHOW server_version;

> **Important:** The `psql` client version and PostgreSQL server version can be different.

---

## 2. 🌐 Connection Information

### Current Connection

Inside `psql`:

    \conninfo

Typical Render output:

    You are connected to database "your_database_name"
    as user "your_database_user"
    on host "your-postgres-host.render.com" at port "5432".
    SSL connection ...

This is useful for confirming that you are connected to the **correct Render database** before running administrative commands.

> **Security:** Do not replace the placeholders above with real production connection details in this public documentation.

### Current Database

    SELECT current_database();

### Current User

    SELECT current_user;

### Current Schema

    SELECT current_schema();

### Show Search Path

    SHOW search_path;

This is particularly useful when working with multiple PostgreSQL schemas.

---

## 3. 📋 List Databases

Inside `psql`:

    \l

More detailed:

    \l+

### Current Database Size

    SELECT pg_size_pretty(
        pg_database_size(current_database())
    );

### Specific Database Size

For documentation/examples, use a placeholder:

    SELECT pg_size_pretty(
        pg_database_size('your_database_name')
    );

Replace `your_database_name` with the actual database name when running the command locally.

---

## 4. 🗂️ Inspect Schemas

### List Schemas

    \dn

More detailed:

    \dn+

### List Tables in Current Schema

    \dt

### List Tables in All Schemas

    \dt *.*

### List Tables in a Specific Schema

    \dt schema_name.*

Example:

    \dt public.*

Another example:

    \dt your_schema.*

---

## 5. 📊 Inspect Tables

### List Tables With Size

    \dt+

### Describe a Table

    \d your_table_name

Example:

    \d users

More detailed:

    \d+ your_table_name

### List Indexes

    \di

### Inspect Indexes for a Specific Table

    SELECT
        indexname,
        indexdef
    FROM pg_indexes
    WHERE tablename = 'your_table_name';

---

## 6. 📏 Measure Database Storage

### Database Size

    SELECT pg_size_pretty(
        pg_database_size(current_database())
    );

### Table Sizes

    SELECT
        relname AS "Table",
        pg_size_pretty(
            pg_total_relation_size(relid)
        ) AS "Total Size"
    FROM pg_catalog.pg_statio_user_tables
    ORDER BY pg_total_relation_size(relid) DESC;

### Table vs Index Size

This version gives a more useful storage breakdown:

    SELECT
        relname AS "Table",
        pg_size_pretty(
            pg_table_size(relid)
        ) AS "Table Size",
        pg_size_pretty(
            pg_indexes_size(relid)
        ) AS "Indexes",
        pg_size_pretty(
            pg_total_relation_size(relid)
        ) AS "Total Size"
    FROM pg_catalog.pg_statio_user_tables
    ORDER BY pg_total_relation_size(relid) DESC;

### What the Sizes Mean

- `pg_table_size()` → table data and related storage, excluding indexes
- `pg_indexes_size()` → indexes belonging to the table
- `pg_total_relation_size()` → table + indexes + associated storage

For database capacity monitoring, **Total Size** is usually the most useful number.

---

## 7. 📊 Inspect Live and Dead Tuples

PostgreSQL uses MVCC (Multi-Version Concurrency Control). Updates and deletes can leave obsolete row versions, commonly called **dead tuples**.

### All Tables

    SELECT
        relname AS "Table",
        n_live_tup AS "Live Tuples",
        n_dead_tup AS "Dead Tuples"
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

### Specific Table

    SELECT
        n_live_tup,
        n_dead_tup
    FROM pg_stat_user_tables
    WHERE relname = 'your_table_name';

### Dead Tuple Ratio

Useful for identifying tables with a relatively large amount of dead-row activity:

    SELECT
        relname AS "Table",
        n_live_tup AS "Live Tuples",
        n_dead_tup AS "Dead Tuples",
        ROUND(
            100.0 * n_dead_tup /
            NULLIF(n_live_tup + n_dead_tup, 0),
            2
        ) AS "Dead %"
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

> `n_live_tup` and `n_dead_tup` are statistics/estimates, not necessarily exact row counts.

---

## 8. 🔎 Detailed Table Statistics

Enable expanded display:

    \x

Then:

    SELECT *
    FROM pg_stat_user_tables
    WHERE relname = 'your_table_name';

To return to normal display:

    \x

### Useful Statistics

Some important columns include:

- `n_live_tup` → estimated live tuples
- `n_dead_tup` → estimated dead tuples
- `last_vacuum` → last manual `VACUUM`
- `last_autovacuum` → last automatic vacuum
- `last_analyze` → last manual `ANALYZE`
- `last_autoanalyze` → last automatic analyze
- `vacuum_count` → number of manual vacuums
- `autovacuum_count` → number of autovacuum operations
- `analyze_count` → number of manual analyzes
- `autoanalyze_count` → number of automatic analyzes

---

## 9. 🧹 VACUUM

`VACUUM` performs routine PostgreSQL maintenance.

It helps PostgreSQL:

- clean up dead tuples
- make space available for reuse
- maintain transaction visibility information
- prevent transaction ID wraparound problems
- improve storage efficiency

### Vacuum a Table

    VACUUM your_table_name;

Example:

    VACUUM users;

### Vacuum and Update Statistics

    VACUUM (ANALYZE) your_table_name;

This performs both:

- `VACUUM`
- `ANALYZE`

This can be useful after significant changes to a table.

### Vacuum the Entire Database

    VACUUM;

Use this carefully on a production system, particularly if the database is large.

---

## 10. 📈 ANALYZE

`ANALYZE` updates PostgreSQL's statistics about table contents.

The query planner uses these statistics to choose efficient execution plans.

### Analyze a Table

    ANALYZE your_table_name;

### Analyze the Entire Database

    ANALYZE;

### Vacuum + Analyze

    VACUUM (ANALYZE) your_table_name;

### Important Difference

`VACUUM` and `ANALYZE` have different purposes:

    VACUUM
      ↓
    Cleans/reclaims dead-row storage for reuse

    ANALYZE
      ↓
    Updates planner statistics

They are complementary, not interchangeable.

---

## 11. ⚠️ VACUUM FULL

`VACUUM FULL` rewrites the table and can return unused space to the operating system.

    VACUUM FULL your_table_name;

### Important Warning

`VACUUM FULL`:

- rewrites the table
- requires an `ACCESS EXCLUSIVE` lock
- blocks normal access to the table while running
- can require significant temporary disk space
- can take considerable time on large tables

Therefore:

> **Do not use `VACUUM FULL` as routine maintenance on a production application.**

Normal `VACUUM` and autovacuum should generally handle routine maintenance.

Use `VACUUM FULL` only when there is a specific reason to reclaim substantial disk space and you can tolerate the locking/downtime implications.

---

## 12. 🤖 Monitoring Autovacuum

PostgreSQL normally performs vacuuming automatically through **autovacuum**.

### Check Whether an Autovacuum Worker Is Running

    SELECT
        pid,
        datname,
        backend_type,
        state,
        query
    FROM pg_stat_activity
    WHERE backend_type = 'autovacuum worker';

> An empty result does **not** mean autovacuum is disabled. It simply means that no autovacuum worker is running at the exact moment of the query.

### Check Autovacuum History

    SELECT
        relname AS "Table",
        last_vacuum,
        last_autovacuum,
        last_analyze,
        last_autoanalyze,
        n_live_tup AS "Live Tuples",
        n_dead_tup AS "Dead Tuples",
        vacuum_count,
        autovacuum_count,
        analyze_count,
        autoanalyze_count
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

This is often more useful than checking only for currently running workers.

It can help answer:

> "Has PostgreSQL been autovacuuming my tables?"

---

## 13. 📊 Monitor VACUUM Progress

PostgreSQL provides `pg_stat_progress_vacuum` for currently running VACUUM operations.

    SELECT
        p.pid,
        a.datname,
        p.relid::regclass AS table_name,
        p.phase,
        p.heap_blks_total,
        p.heap_blks_scanned,
        p.heap_blks_vacuumed
    FROM pg_stat_progress_vacuum p
    JOIN pg_stat_activity a
        ON a.pid = p.pid;

### More Detailed Version

    SELECT
        p.pid,
        a.datname,
        p.relid::regclass AS table_name,
        p.phase,
        p.heap_blks_total,
        p.heap_blks_scanned,
        p.heap_blks_vacuumed,
        p.index_vacuum_count,
        p.num_dead_tuples,
        p.max_dead_tuples
    FROM pg_stat_progress_vacuum p
    JOIN pg_stat_activity a
        ON a.pid = p.pid;

### Important

`pg_stat_progress_vacuum` shows VACUUM operations that are **currently in progress**.

If the query returns no rows, there may simply be no VACUUM running at that moment.

---

## 14. 🔄 Autovacuum vs Manual VACUUM

### Autovacuum

PostgreSQL automatically decides when to vacuum tables based on table activity and configuration.

    INSERT / UPDATE / DELETE
            ↓
    Dead tuples accumulate
            ↓
    Autovacuum threshold reached
            ↓
    Autovacuum worker starts
            ↓
    VACUUM

### Manual VACUUM

You explicitly request maintenance:

    VACUUM your_table_name;

For normal production operation:

> **Prefer PostgreSQL's autovacuum rather than manually vacuuming tables on a fixed schedule unless there is a specific operational reason.**

---

## 15. 🔌 Monitor Active Connections

### Current Connections

    SELECT
        count(*)
    FROM pg_stat_activity;

### Connections by Database

    SELECT
        datname,
        count(*) AS connections
    FROM pg_stat_activity
    GROUP BY datname
    ORDER BY connections DESC;

### Detailed Active Connections

    SELECT
        pid,
        usename,
        datname,
        client_addr,
        application_name,
        state,
        backend_start,
        query_start,
        query
    FROM pg_stat_activity
    ORDER BY query_start DESC NULLS LAST;

### Only Active Queries

    SELECT
        pid,
        usename,
        datname,
        application_name,
        state,
        query_start,
        query
    FROM pg_stat_activity
    WHERE state = 'active'
    ORDER BY query_start;

---

## 16. ⏱️ Find Long-Running Queries

    SELECT
        pid,
        usename,
        datname,
        now() - query_start AS duration,
        state,
        query
    FROM pg_stat_activity
    WHERE state <> 'idle'
    ORDER BY duration DESC;

### Queries Running Longer Than 1 Minute

    SELECT
        pid,
        usename,
        datname,
        now() - query_start AS duration,
        state,
        query
    FROM pg_stat_activity
    WHERE state <> 'idle'
      AND now() - query_start > interval '1 minute'
    ORDER BY duration DESC;

---

## 17. 🔒 Find Blocking Queries

A blocked query can make an application appear slow even when CPU and database storage look normal.

### Find Waiting/Blocked Sessions

    SELECT
        pid,
        usename,
        datname,
        wait_event_type,
        wait_event,
        state,
        query
    FROM pg_stat_activity
    WHERE wait_event IS NOT NULL;

### Identify Blocking Relationships

    SELECT
        blocked.pid AS blocked_pid,
        blocked.query AS blocked_query,
        blocking.pid AS blocking_pid,
        blocking.query AS blocking_query
    FROM pg_stat_activity blocked
    JOIN pg_stat_activity blocking
        ON blocking.pid = ANY(
            pg_blocking_pids(blocked.pid)
        );

This is particularly useful when a Rails application appears to hang during database operations.

---

## 18. 🧮 Find the Largest Tables

    SELECT
        schemaname,
        relname AS table_name,
        pg_size_pretty(
            pg_total_relation_size(relid)
        ) AS total_size
    FROM pg_stat_user_tables
    ORDER BY pg_total_relation_size(relid) DESC;

### Include Index Size

    SELECT
        schemaname,
        relname AS table_name,
        pg_size_pretty(
            pg_table_size(relid)
        ) AS table_size,
        pg_size_pretty(
            pg_indexes_size(relid)
        ) AS index_size,
        pg_size_pretty(
            pg_total_relation_size(relid)
        ) AS total_size
    FROM pg_stat_user_tables
    ORDER BY pg_total_relation_size(relid) DESC;

---

## 19. 📑 Inspect Index Usage

PostgreSQL tracks index usage statistics.

    SELECT
        schemaname,
        relname AS table_name,
        indexrelname AS index_name,
        idx_scan,
        pg_size_pretty(
            pg_relation_size(indexrelid)
        ) AS index_size
    FROM pg_stat_user_indexes
    ORDER BY idx_scan ASC;

### Important Warning

An index with low `idx_scan` is **not automatically safe to delete**.

Some indexes may be:

- required by constraints
- used infrequently but critically
- needed for specific reporting queries
- used only during periodic jobs

Always investigate before removing an index.

---

## 20. 🧪 Check Table Statistics After Maintenance

    SELECT
        relname AS "Table",
        n_live_tup AS "Live",
        n_dead_tup AS "Dead",
        last_vacuum,
        last_autovacuum,
        last_analyze,
        last_autoanalyze
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

This is a useful before/after query when investigating vacuum behavior.

---

## 21. 🛠️ Useful `psql` Commands

These are `psql` meta-commands, not SQL statements.

### Help

    \?

### SQL Help

    \h

Example:

    \h VACUUM

### List Databases

    \l

### List Schemas

    \dn

### List Tables

    \dt

### List All Tables

    \dt *.*

### Describe Table

    \d table_name

### Describe Table in Detail

    \d+ table_name

### List Indexes

    \di

### Show Current Connection

    \conninfo

### Show Query History

    \s

### Clear Screen

    \! clear

### Quit `psql`

    \q

---

## 22. 🔐 Render PostgreSQL Connection Check

Before running maintenance commands against a Render database, verify the connection:

    \conninfo

A safe example of the expected information is:

    You are connected to database "your_database_name"
    as user "your_database_user"
    on host "your-postgres-host.render.com" at port "5432".

Then verify the database:

    SELECT current_database();

Then verify the user:

    SELECT current_user;

This is especially important when multiple Rails applications share one PostgreSQL instance.

> **Never commit the real output of `\conninfo` to a public repository if it contains your actual Render hostname or other infrastructure details.**

---

## 23. 🧭 Recommended Diagnostic Workflow

When investigating PostgreSQL performance or storage issues, avoid immediately running `VACUUM FULL`.

Use this sequence instead.

### Step 1 — Confirm Connection

    \conninfo

### Step 2 — Check Database Size

    SELECT pg_size_pretty(
        pg_database_size(current_database())
    );

### Step 3 — Find Large Tables

    SELECT
        schemaname,
        relname AS table_name,
        pg_size_pretty(
            pg_total_relation_size(relid)
        ) AS total_size
    FROM pg_stat_user_tables
    ORDER BY pg_total_relation_size(relid) DESC;

### Step 4 — Check Dead Tuples

    SELECT
        relname AS "Table",
        n_live_tup AS "Live Tuples",
        n_dead_tup AS "Dead Tuples"
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

### Step 5 — Check Autovacuum History

    SELECT
        relname AS "Table",
        last_autovacuum,
        autovacuum_count,
        n_dead_tup
    FROM pg_stat_user_tables
    ORDER BY n_dead_tup DESC;

### Step 6 — Check Currently Running Autovacuum

    SELECT
        pid,
        datname,
        backend_type,
        state,
        query
    FROM pg_stat_activity
    WHERE backend_type = 'autovacuum worker';

### Step 7 — Check Current VACUUM Progress

    SELECT
        p.pid,
        a.datname,
        p.relid::regclass AS table_name,
        p.phase,
        p.heap_blks_total,
        p.heap_blks_scanned,
        p.heap_blks_vacuumed
    FROM pg_stat_progress_vacuum p
    JOIN pg_stat_activity a
        ON a.pid = p.pid;

### Step 8 — Check Long-Running Queries

    SELECT
        pid,
        usename,
        datname,
        now() - query_start AS duration,
        state,
        query
    FROM pg_stat_activity
    WHERE state <> 'idle'
    ORDER BY duration DESC;

### Step 9 — Check Blocking Queries

    SELECT
        blocked.pid AS blocked_pid,
        blocked.query AS blocked_query,
        blocking.pid AS blocking_pid,
        blocking.query AS blocking_query
    FROM pg_stat_activity blocked
    JOIN pg_stat_activity blocking
        ON blocking.pid = ANY(
            pg_blocking_pids(blocked.pid)
        );

Only after understanding the situation should you consider manual maintenance.

---

## 24. ⚠️ Commands Requiring Extra Care

The following commands can have significant production impact:

    VACUUM FULL;

    REINDEX DATABASE database_name;

    DROP TABLE table_name;

    TRUNCATE TABLE table_name;

    ALTER TABLE ...;

Do not run these simply because a table has dead tuples or because a database appears slow.

First determine the actual cause.

---

## 25. 🧠 Important PostgreSQL Concepts

### Dead Tuples

Updates and deletes can leave obsolete row versions because PostgreSQL uses MVCC.

    UPDATE / DELETE
          ↓
    Old row version becomes obsolete
          ↓
    Dead tuple
          ↓
    VACUUM
          ↓
    Space becomes reusable

### VACUUM

Primarily handles dead-row cleanup and makes storage reusable.

### VACUUM FULL

Rewrites the table to compact it and can return unused space to the operating system, but requires a strong table lock.

### ANALYZE

Updates statistics used by the PostgreSQL query planner.

### Autovacuum

Automatically performs maintenance based on table activity and PostgreSQL configuration.

### `pg_stat_activity`

Shows current database sessions and queries.

### `pg_stat_progress_vacuum`

Shows progress of currently running VACUUM operations.

### `pg_stat_user_tables`

Provides table-level activity and maintenance statistics.

---

## 26. 🚀 Quick Reference

| Task | Command |
|---|---|
| PostgreSQL client version | `psql --version` |
| PostgreSQL server version | `SELECT version();` |
| Connection information | `\conninfo` |
| Current database | `SELECT current_database();` |
| Current user | `SELECT current_user;` |
| List databases | `\l+` |
| List schemas | `\dn` |
| List tables | `\dt` |
| Describe table | `\d table_name` |
| Database size | `pg_database_size()` |
| Table size | `pg_total_relation_size()` |
| Live/dead tuples | `pg_stat_user_tables` |
| Manual vacuum | `VACUUM table_name;` |
| Vacuum + statistics | `VACUUM (ANALYZE) table_name;` |
| Update statistics | `ANALYZE table_name;` |
| Compact table | `VACUUM FULL table_name;` ⚠️ |
| Check autovacuum workers | `pg_stat_activity` |
| Vacuum progress | `pg_stat_progress_vacuum` |
| Active connections | `pg_stat_activity` |
| Long-running queries | `pg_stat_activity` |
| Blocking queries | `pg_blocking_pids()` |
| Index usage | `pg_stat_user_indexes` |

---

## 27. 📝 Notes for Public GitHub Repository

- Use placeholders such as `your_database_name`, `your_database_user`, and `your-postgres-host.render.com` in public documentation.
- Never commit database passwords.
- Never commit a real `DATABASE_URL` containing credentials.
- Never commit `.env` files containing production secrets.
- Never commit Rails `config/master.key` or production credentials containing secrets.
- Never commit API keys, SMTP passwords, Render API tokens, Cloudflare tokens, GitHub tokens, or similar secrets.
- A database name alone is generally not a credential and does not provide database access.
- A database username alone is generally not sufficient to access the database.
- A real database hostname is not equivalent to a password, but it is usually better to avoid publishing unnecessary infrastructure details.
- If credentials are accidentally committed, removing the file in a later commit is **not sufficient** because Git history may still contain the secret. Rotate the exposed credential immediately.
- Before pushing a public repository, review the Git diff and repository history for accidentally exposed secrets.

---

## 28. 📝 Notes for Render PostgreSQL

- Always run `\conninfo` before performing administrative work.
- Verify that you are connected to the intended Render database.
- `psql --version` reports the **client** version; `SELECT version()` reports the **server** version.
- `n_live_tup` and `n_dead_tup` are statistics and should not be treated as exact row counts.
- An empty `pg_stat_progress_vacuum` result simply means no VACUUM is currently running.
- An empty autovacuum-worker query does not mean autovacuum is disabled.
- Normal `VACUUM` generally does not require an exclusive table lock.
- `VACUUM FULL` requires an `ACCESS EXCLUSIVE` lock and can block application traffic.
- Do not use `VACUUM FULL` as routine maintenance.
- Autovacuum should normally handle routine vacuuming automatically.
- `ANALYZE` is about query-planner statistics, not dead-tuple cleanup.
- Large indexes can consume significant database storage.
- Low index usage does not automatically mean an index should be deleted.
- When multiple Rails applications share one PostgreSQL instance, monitor database size, connections, large tables, indexes, and autovacuum behavior at the database level.
- For production databases, investigate the cause of a performance/storage problem before applying disruptive maintenance commands.

---

## 29. 🎯 Practical Rule of Thumb

For routine PostgreSQL administration:

    Monitor
      ↓
    Measure
      ↓
    Understand the problem
      ↓
    Check autovacuum/statistics
      ↓
    Use VACUUM / ANALYZE when appropriate
      ↓
    Only use disruptive operations when justified

In particular:

    Dead tuples
        ↓
    Usually → Autovacuum
        ↓
    If necessary → VACUUM
        ↓
    If planner statistics are stale → ANALYZE
        ↓
    If severe table bloat requires physical compaction
        → Consider VACUUM FULL carefully

**Do not treat `VACUUM FULL` as the normal solution to dead tuples.**