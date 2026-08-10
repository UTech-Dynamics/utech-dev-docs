# Engineering Playbook: PostgreSQL 18 Disk Space & Slot Reuse

This reference note explains how PostgreSQL 18 (including instances hosted on Render.com) handles deleted data, how empty space slots are reused, and how to safely optimize disk storage without risking data loss.

---

## 1. Core Mechanics: How PostgreSQL Deletes Data

* **Marked, Not Erased:** When you execute a `DELETE` command, PostgreSQL 18 does not physically erase the data or shrink the database file. It marks the affected rows as **Dead Tuples** (invisible to your application).
* **The Free Space Map (FSM):** The physical "shells" or slots left behind by dead tuples are tracked by PostgreSQL's internal directory, the Free Space Map.
* **Slot Reuse:** When new data is inserted, PostgreSQL scans the FSM and writes the new records directly into those empty slots. 

### 💡 Pro-Tip for Postgres 18: Use `uuidv7()`
If your table uses UUIDs as primary keys, ensure you migrate to the native **`uuidv7()`** function introduced in PostgreSQL 18. Because UUIDv7 generates timestamp-ordered IDs, new inserts will naturally pack sequentially into recently emptied slots, drastically minimizing page fragmentation compared to old random `uuidv4` patterns.

---

## 2. Automated Maintenance: `autovacuum`

By default, Render.com and modern PostgreSQL setups run a background daemon called **`autovacuum`**. 

* **What it does:** It periodically cleans up dead tuples and officially marks their slots as "available for reuse" in the FSM. 
* **Postgres 18 Advantage:** Thanks to the architectural introduction of the Asynchronous I/O (AIO) subsystem in v18, background vacuum operations have a much lower impact on live transaction latency while accelerating sequential scans.
* **The Catch:** Standard `autovacuum` **does not** return disk space to the operating system. Your total database file size on disk will not shrink, but it will stop growing because new data fills the old slots first.

---

## 3. Essential Commands & Safe Execution

Use these strategies depending on your operational goals. **Always test commands in a staging environment before running them in production.**

### Scenario A: Reclaiming Disk Space Safely (Zero Downtime)
If you want to ensure empty slots are instantly available for *new data* but cannot afford to lock your application.
```sql
-- Safely cleans up dead tuples in a specific table or the whole DB.
-- Your application can read and write data normally while this runs.
VACUUM VERBOSE your_table_name;
```

### Scenario B: Shrinking Database Size (Requires Downtime)
If you deleted massive amounts of data (e.g., millions of rows) and need to shrink the actual disk footprint to avoid Render storage fees or limits.
```sql
-- CRITICAL WARNING: This packs data tightly and returns disk space to the OS.
-- It places an EXCLUSIVE LOCK on the table. 
-- Your application CANNOT read or write data until this finishes.
VACUUM FULL VERBOSE your_table_name;
```

### Scenario C: Wiping a Whole Table Safely (Instant Space Return)
If you want to clear out 100% of a table's data, do not use `DELETE`. Use `TRUNCATE`.
```sql
-- Instantly drops the data and returns the physical disk space to the OS.
-- Fast, efficient, and avoids creating millions of dead tuples.
TRUNCATE TABLE your_table_name;
```

---

## 4. Operational Best Practices to Prevent Data Loss

1. **Never kill a running `VACUUM FULL` abruptly:** It creates a temporary copy of the table to pack the data. Forcing it to stop mid-process can leave orphaned files and temporarily consume double the disk space.
2. **Monitor Disk Space Before `VACUUM FULL`:** Because `VACUUM FULL` creates a new, compacted copy of your table before deleting the old one, you must have at least **equal to or greater than** the table's current size in free disk space to run it.
3. **Always take a snapshot backup first:** Before running heavy maintenance like `VACUUM FULL` or `TRUNCATE`, trigger a manual backup in your Render.com dashboard.
