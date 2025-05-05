**Query optimization in Rails** is essential for building high-performance applications, especially when scaling to large datasets. Here’s a **practical guide** to optimizing database queries in Rails using **ActiveRecord**, indexing, and query design.

---

## 🚦 1. **Avoid N+1 Queries**

**Problem:**

```ruby
Post.all.each do |post|
  puts post.comments.count
end
```

**Solution:**

Use **`.includes`** to eager-load associations:

```ruby
Post.includes(:comments).each do |post|
  puts post.comments.count
end
```

🔍 Use the [**Bullet** gem](https://github.com/flyerhzm/bullet) to automatically detect N+1 issues in development.

---

## 🧠 2. **Use `select` to Load Only Needed Columns**

```ruby
# Loads all columns (slow for large tables)
User.all

# Loads only needed data (faster)
User.select(:id, :email).where(active: true)
```

---

## 📦 3. **Use Batching for Large Data**

```ruby
# Avoid this
User.all.each { |user| process(user) }

# Use this
User.find_each(batch_size: 1000) { |user| process(user) }
```

---

## ⏳ 4. **Use `limit`, `offset`, and pagination**

```ruby
User.limit(10).offset(20)
```

Use a pagination gem:

* `kaminari`
* `pagy` (very fast)
* `will_paginate`

---

## 📊 5. **Add Proper Indexes**

Add indexes on:

* `foreign keys`
* `columns used in WHERE`, `ORDER`, or `JOIN` clauses

Migration:

```ruby
add_index :users, :email, unique: true
add_index :orders, :user_id
```

Check slow queries with tools like:

* `EXPLAIN` in SQL
* PostgreSQL’s `pg_stat_statements`

---

## 🧰 6. **Use Scopes & Query Caching**

Define reusable scopes:

```ruby
scope :active, -> { where(active: true) }
scope :recent, -> { order(created_at: :desc).limit(5) }
```

Use `Rails.cache.fetch` for expensive queries:

```ruby
Rails.cache.fetch("top_products", expires_in: 10.minutes) do
  Product.order(sales: :desc).limit(10).to_a
end
```

---

## 🔍 7. **Reduce DB Calls with `pluck` or `exists?`**

```ruby
# Bad: loads full record
User.where(email: 'x@example.com').first.present?

# Good
User.exists?(email: 'x@example.com')

# Good: fetch array of values
User.where(active: true).pluck(:id)
```

---

## 🔄 8. **Use `joins` Instead of `includes` for Filtering**

```ruby
# eager loads comments but filters at Ruby level (slow)
Post.includes(:comments).where(comments: { visible: true })

# better for DB-level filtering
Post.joins(:comments).where(comments: { visible: true })
```

---

## 🧼 9. **Analyze Queries with `.to_sql`**

```ruby
puts User.where(active: true).order(:created_at).to_sql
```

Use this to debug or copy queries into `psql` or `EXPLAIN`.

---

## ✅ 10. **Advanced Tips**

* Use `materialized views` or `read replicas` for heavy reports
* Use **counter caches** for `.count` performance:

```bash
rails generate migration AddCommentsCountToPosts comments_count:integer
```

```ruby
belongs_to :post, counter_cache: true
```

* Use `raw SQL` only when needed, wrapped safely:

```ruby
User.find_by_sql("SELECT * FROM users WHERE active = true LIMIT 5")
```

---

Would you like me to review a slow query or log and help optimize it?
