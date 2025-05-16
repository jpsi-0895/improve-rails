In **Ruby on Rails**, the `where` method is used in **Active Record queries** to filter records from the database by specified conditions.

---

### ✅ Basic Syntax

```ruby
Model.where(condition)
```

It returns an **ActiveRecord::Relation** (a chainable query object), not a plain array.

---

### 🔹 Examples

#### 1. **Simple equality**

```ruby
User.where(name: "Alice")
```

SQL equivalent:

```sql
SELECT * FROM users WHERE name = 'Alice';
```

---

#### 2. **Multiple conditions**

```ruby
User.where(name: "Alice", age: 30)
```

SQL:

```sql
SELECT * FROM users WHERE name = 'Alice' AND age = 30;
```

---

#### 3. **Using operators**

```ruby
User.where("age > ?", 25)
```

SQL:

```sql
SELECT * FROM users WHERE age > 25;
```

This uses **parameter binding** (`?`) to prevent SQL injection.

---

#### 4. **IN clause**

```ruby
User.where(id: [1, 2, 3])
```

SQL:

```sql
SELECT * FROM users WHERE id IN (1, 2, 3);
```

---

#### 5. **NOT condition**

```ruby
User.where.not(name: "Bob")
```

SQL:

```sql
SELECT * FROM users WHERE name != 'Bob';
```

---

#### 6. **Combining with `or`**

```ruby
User.where(name: "Alice").or(User.where(age: 25))
```

SQL:

```sql
SELECT * FROM users WHERE name = 'Alice' OR age = 25;
```

---

### 🔁 Chaining Queries

Because `where` returns a relation, you can chain other methods like `order`, `limit`, `joins`, etc.

```ruby
User.where(active: true).order(:created_at).limit(10)
```

---

Would you like an example with associations or scopes too?
