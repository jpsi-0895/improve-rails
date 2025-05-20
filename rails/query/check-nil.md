To check if a `category` object is **empty or not** in Rails, your approach depends on **what `category` is**:

---

### 🧩 1. **If `category` is an ActiveRecord object**

```ruby
if category.present?
  # category exists and is not nil
else
  # category is nil or blank
end
```

* `present?` = not `nil` and not empty
* `blank?` = `nil`, empty string, or empty collection

#### ✅ Example:

```ruby
category = Category.find_by(slug: params[:slug])

if category.present?
  # Do something with the category
else
  # Handle case where category is nil (not found)
end
```

---

### 📦 2. **If `category` is a collection (like `@category.blogs`)**

```ruby
if category.any?
  # At least one record is present
end
```

Or:

```ruby
if category.empty?
  # No associated records
end
```

Or more idiomatically:

```ruby
if category.blank?
  # category is nil or has no records
end
```

---

### 🛡️ 3. **Safe Navigation (avoid `nil` errors)**

```ruby
if category&.blogs&.any?
  # Category exists and has blogs
else
  # Either category is nil, or no blogs
end
```

---

### 🧠 Summary

| Check                   | Meaning                         |
| ----------------------- | ------------------------------- |
| `category.nil?`         | Exactly nil                     |
| `category.blank?`       | `nil` or empty                  |
| `category.present?`     | Not nil and not empty           |
| `category.any?`         | Collection has items            |
| `category.empty?`       | Collection has no items         |
| `category&.blogs&.any?` | Safe check for associated blogs |

---

Let me know exactly what `category` refers to (a record, a collection, or association) and I’ll give you the most appropriate condition.
