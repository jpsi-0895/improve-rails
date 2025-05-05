In **Rails**, ORM (Object-Relational Mapping) is handled by **ActiveRecord**, which provides a rich set of **query functions** to interact with the database in a Ruby-like way.

Here’s a comprehensive cheat sheet of **common ORM query functions** in Rails:

---

## 🔍 Basic Retrieval

| Function          | Example                            | Description                       |
| ----------------- | ---------------------------------- | --------------------------------- |
| `.all`            | `Product.all`                      | Get all records                   |
| `.first`, `.last` | `User.first`, `User.last`          | Get first/last record             |
| `.find`           | `User.find(1)`                     | Find by ID (raises if not found)  |
| `.find_by`        | `User.find_by(email: 'x@y.com')`   | Find first match (returns `nil`)  |
| `.where`          | `Product.where(category: 'Books')` | Filter records (returns Relation) |
| `.pluck`          | `User.pluck(:email)`               | Get array of column values        |
| `.select`         | `User.select(:id, :name)`          | Select specific columns           |

---

## 📐 Conditions and Comparisons

```ruby
User.where("age > ?", 18)
User.where.not(status: 'archived')
Product.where(price: 10..20)
```

---

## 🔄 Ordering, Limiting, Offsetting

```ruby
Product.order(created_at: :desc)
Product.limit(5)
Product.offset(10)
```

---

## 🔁 Joins and Includes (Eager Loading)

```ruby
# Eager loading to avoid N+1
Post.includes(:comments).where(status: 'published')

# SQL join
Post.joins(:user).where(users: { active: true })
```

---

## 🧮 Aggregates

```ruby
User.count
Product.average(:price)
Order.sum(:total)
User.maximum(:age)
```

---

## 🔎 Scopes

In model:

```ruby
scope :active, -> { where(active: true) }
scope :recent, -> { order(created_at: :desc).limit(5) }
```

Usage:

```ruby
User.active.recent
```

---

## 🧱 Grouping and Having

```ruby
Order.group(:user_id).sum(:total)
Product.group(:category).count
```

---

## ⚠️ Query Debugging Tip

Enable SQL logs in development or use:

```ruby
puts User.where(active: true).to_sql
```

---

## ✅ Chainable

ActiveRecord queries are chainable:

```ruby
User.where(active: true).order(created_at: :desc).limit(10)
```

---

Would you like examples specific to your models (like `Product`, `Order`, or `User`)?
