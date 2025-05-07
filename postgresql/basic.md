Here’s a **PostgreSQL interview cheat sheet** that covers the most commonly asked SQL concepts, commands, and sample questions tailored for technical interviews:

---

## 🧠 **PostgreSQL Interview Topics & Questions**

### 🔹 **1. Basic SQL & Queries**

* **SELECT**, **WHERE**, **ORDER BY**, **GROUP BY**, **HAVING**, **LIMIT**, **OFFSET**

**Q1:** Get the top 5 highest-paid employees.

```sql
SELECT name, salary FROM employees
ORDER BY salary DESC
LIMIT 5;
```

**Q2:** Find the number of employees in each department.

```sql
SELECT department_id, COUNT(*) 
FROM employees
GROUP BY department_id;
```

---

### 🔹 **2. Joins (INNER, LEFT, RIGHT, FULL)**

**Q3:** Get all employees and their manager's name.

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

### 🔹 **3. Subqueries & CTEs**

**Q4:** Get employees who earn more than the average salary.

```sql
SELECT name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Q5:** Use a CTE to get department with highest total salary.

```sql
WITH dept_salaries AS (
  SELECT department_id, SUM(salary) AS total_salary
  FROM employees
  GROUP BY department_id
)
SELECT department_id, total_salary
FROM dept_salaries
ORDER BY total_salary DESC
LIMIT 1;
```

---

### 🔹 **4. Window Functions**

**Q6:** Rank employees by salary within their departments.

```sql
SELECT name, department_id, salary,
       RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
FROM employees;
```

---

### 🔹 **5. Data Definition Language (DDL)**

* **CREATE**, **ALTER**, **DROP**

**Q7:** Create a table:

```sql
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  salary NUMERIC,
  department_id INT,
  manager_id INT
);
```

---

### 🔹 **6. Indexes and Performance**

**Q8:** How do you create an index?

```sql
CREATE INDEX idx_employee_salary ON employees(salary);
```

**Follow-up:** Explain when to use an index (e.g., for frequent filtering, sorting).

---

### 🔹 **7. Transactions and Concurrency**

* **BEGIN**, **COMMIT**, **ROLLBACK**
* ACID properties

**Q9:** What does SERIALIZABLE isolation do?

> It ensures the strictest transaction isolation – transactions appear to execute one at a time.

---

### 🔹 **8. JSON & PostgreSQL Extensions**

**Q10:** How do you query a JSON column?

```sql
SELECT data->>'name' AS name
FROM users
WHERE data->>'active' = 'true';
```

---

## 🔧 Advanced Concepts (For Senior Roles)

* Index types (B-Tree, GIN, GiST)
* Table partitioning
* Vacuuming and autovacuum
* Explain plans and query optimization

---

Would you like a downloadable cheat sheet or practice questions with sample data?
