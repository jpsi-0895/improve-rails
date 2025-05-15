Sure! Here's the **SQL statement** to create a table with the two specified fields:

* `page_type`: `ENUM` type with a default value
* `country_name`: `VARCHAR`

---

### ✅ SQL: Create Table Example

```sql
CREATE TABLE pages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    page_type ENUM('home', 'about', 'contact', 'blog') DEFAULT 'home',
    country_name VARCHAR(100)
);
```

---

### 🧠 Notes:

* You can change the `ENUM` values to whatever options you need.
* `DEFAULT 'home'` sets the default value for `page_type`.
* `VARCHAR(100)` gives enough space for most country names.
* `id` is included as a typical primary key (you can remove it if unnecessary).

---

### ❗ If you're **modifying an existing table**, use:

```sql
ALTER TABLE your_table_name
ADD COLUMN page_type ENUM('home', 'about', 'contact', 'blog') DEFAULT 'home',
ADD COLUMN country_name VARCHAR(100);
```

---

Would you like help:

* Generating dummy data for this table?
* Adding constraints like `NOT NULL` or `UNIQUE`?
* Including this in a migration file?

Let me know!
