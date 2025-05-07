### 🧱 PostgreSQL Table Partitioning — Complete Guide

**Table partitioning** in PostgreSQL is a way to divide large tables into smaller, more manageable pieces (partitions), while still being treated as a single table in queries.

---

## 🚀 Why Use Partitioning?

* Improve query performance on large datasets.
* Speed up **bulk inserts/deletes**.
* Simplify **data lifecycle management** (e.g., archiving old data).
* Reduce **index sizes** per partition.

---

## 🔧 Partitioning Methods in PostgreSQL

| Type      | Description                                      | Best For                      |
| --------- | ------------------------------------------------ | ----------------------------- |
| **Range** | Based on a range of values (e.g., dates, IDs)    | Time-series or sequential IDs |
| **List**  | Based on discrete values (e.g., country, status) | Categorical data              |
| **Hash**  | Based on a hash function                         | Uniform distribution needed   |

---

## 📘 Example: Range Partitioning by Date

### 1. Create the Parent Table

```sql
CREATE TABLE sales (
  id SERIAL,
  sale_date DATE NOT NULL,
  amount NUMERIC,
  region TEXT
) PARTITION BY RANGE (sale_date);
```

---

### 2. Create Child Partitions

```sql
CREATE TABLE sales_2023_q1 PARTITION OF sales
  FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');

CREATE TABLE sales_2023_q2 PARTITION OF sales
  FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');
```

---

### 3. Insert Data (PostgreSQL routes it automatically)

```sql
INSERT INTO sales (sale_date, amount, region)
VALUES ('2023-02-15', 500, 'US');
```

---

## 📘 Example: List Partitioning by Region

```sql
CREATE TABLE users (
  id SERIAL,
  name TEXT,
  country TEXT NOT NULL
) PARTITION BY LIST (country);

CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
CREATE TABLE users_uk PARTITION OF users FOR VALUES IN ('UK');
```

---

## 📘 Example: Hash Partitioning (e.g., Sharding by User ID)

```sql
CREATE TABLE logs (
  id SERIAL,
  user_id INT,
  event TEXT
) PARTITION BY HASH (user_id);

CREATE TABLE logs_p0 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE logs_p1 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 1);
-- and so on...
```

---

## ✅ Best Practices

* Always index the **partition key**.
* Use **`CHECK` constraints** or **DEFAULT** partitions for safety.
* Combine with **pg\_partman** for automated partition maintenance.
* Avoid **cross-partition joins** for performance.

---

Would you like a diagram or a real-world use case like partitioning logs, events, or sales data?
