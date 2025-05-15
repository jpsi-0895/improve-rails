To get **MySQL table field (column) information**, you can use several useful SQL commands or queries depending on what exactly you want to inspect — data types, keys, defaults, etc.

---

## 🔍 Basic: Get All Column Info (Structure)

```sql
DESCRIBE your_table_name;
```

### Example:

```sql
DESCRIBE users;
```

### Output:

| Field | Type         | Null | Key | Default | Extra           |
| ----- | ------------ | ---- | --- | ------- | --------------- |
| id    | int(11)      | NO   | PRI | NULL    | auto\_increment |
| name  | varchar(100) | YES  |     | NULL    |                 |
| email | varchar(100) | NO   | UNI | NULL    |                 |

---

## 🛠 Detailed Info (via INFORMATION\_SCHEMA)

To get more detailed info, including collation, default values, comments, etc.:

```sql
SELECT 
  COLUMN_NAME,
  COLUMN_TYPE,
  IS_NULLABLE,
  COLUMN_KEY,
  COLUMN_DEFAULT,
  EXTRA,
  COLUMN_COMMENT
FROM 
  INFORMATION_SCHEMA.COLUMNS
WHERE 
  TABLE_SCHEMA = 'your_database_name'
  AND TABLE_NAME = 'your_table_name';
```

---

## 🔑 Show Keys and Indexes

```sql
SHOW INDEX FROM your_table_name;
```

This shows:

* Primary and secondary indexes
* Unique constraints
* Column names and index types

---

## 🔄 Show Table Create Statement

To see how the table was created (including field definitions, constraints, etc.):

```sql
SHOW CREATE TABLE your_table_name;
```

---

## ✅ Summary Table

| Task                     | Command                         |
| ------------------------ | ------------------------------- |
| Basic field info         | `DESCRIBE table_name;`          |
| Detailed column metadata | `INFORMATION_SCHEMA.COLUMNS`    |
| Show keys and indexes    | `SHOW INDEX FROM table_name;`   |
| Full create statement    | `SHOW CREATE TABLE table_name;` |

---

Would you like:

* A script to export column info as JSON or CSV?
* A web-based UI (e.g. in PHP or Node) to view table schemas?
* Sample table field definitions?

Just let me know!


ALTER TABLE pages
ADD COLUMN page_type ENUM('product', 'country') DEFAULT 'product',
ADD COLUMN country_name VARCHAR(200);