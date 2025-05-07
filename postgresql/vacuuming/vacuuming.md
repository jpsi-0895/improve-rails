### 🧹 PostgreSQL Vacuuming & Autovacuum — Explained

In PostgreSQL, **VACUUM** is essential for cleaning up dead rows and maintaining database performance and storage efficiency. Without it, your tables can bloat and slow down significantly over time.

---

## 🔍 Why Does PostgreSQL Need Vacuuming?

PostgreSQL uses **MVCC (Multi-Version Concurrency Control)** — instead of updating rows in place, it creates new row versions and marks old ones as dead.

These dead rows:

* Waste space
* Slow down queries
* Prevent reusing table space

👉 **VACUUM** clears them out.

---

## 🧼 Types of Vacuum

| Type             | Description                                                    | Use Case                                   |
| ---------------- | -------------------------------------------------------------- | ------------------------------------------ |
| `VACUUM`         | Removes dead row versions, but does not return space to the OS | Routine cleanup                            |
| `VACUUM FULL`    | Compacts table and returns space to OS (exclusive lock)        | Major cleanup (e.g., after massive delete) |
| `ANALYZE`        | Updates query planner statistics                               | Should be run regularly                    |
| `VACUUM ANALYZE` | Does both                                                      | Good default maintenance                   |

---

## ⚙️ What Is Autovacuum?

**Autovacuum** is a **background process** that automatically runs `VACUUM` and `ANALYZE` on tables based on certain thresholds.

### ✅ Benefits:

* Keeps tables clean and performant
* Requires minimal manual intervention

### 🔧 Key Parameters (in `postgresql.conf`)

| Parameter                        | Default | Description                               |
| -------------------------------- | ------- | ----------------------------------------- |
| `autovacuum`                     | `on`    | Enables autovacuum                        |
| `autovacuum_vacuum_threshold`    | `50`    | Minimum dead tuples to trigger            |
| `autovacuum_vacuum_scale_factor` | `0.2`   | Multiplied by table size to set threshold |
| `autovacuum_naptime`             | `1min`  | Delay between autovacuum runs             |

#### 📘 Example:

For a table with 10,000 rows:

```
Threshold = 50 + (0.2 * 10,000) = 2,050 dead rows
```

---

## 🛠 Manual Vacuum Command

```sql
VACUUM;                -- Basic cleanup
VACUUM FULL;           -- Reclaims space, locks table
VACUUM ANALYZE;        -- Cleans and updates statistics
```

---

## 🧠 Best Practices

* Don’t disable autovacuum unless you have a replacement strategy.

* Monitor with:

  ```sql
  SELECT relname, n_dead_tup
  FROM pg_stat_user_tables
  ORDER BY n_dead_tup DESC;
  ```

* Use `pg_stat_activity` and `pg_stat_all_tables` to track activity and health.

* Run `VACUUM FULL` during low-traffic times if bloat is severe.

---

Would you like help checking autovacuum settings on your own PostgreSQL server or analyzing bloat?
