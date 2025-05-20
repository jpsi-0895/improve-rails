Handling `nil` records in Rails (or Ruby) is essential to avoid `NoMethodError` or application crashes. Here's how to handle `nil` values safely and idiomatically in various scenarios:

---

### 🔍 1. **Check if a record is `nil`**

```ruby
if record.nil?
  # Handle the nil case
else
  # Safe to use the record
end
```

Or more succinctly:

```ruby
if record
  # Proceed
else
  # Handle nil
end
```

---

### 🛡️ 2. **Use Safe Navigation Operator (`&.`)**

If a record might be `nil` but you want to safely call a method on it:

```ruby
record&.name
# Returns nil instead of throwing an error if record is nil
```

---

### 🧱 3. **Use `try` Method**

```ruby
record.try(:name)
# Similar to &., works on older Ruby versions too
```

Note: `try` is more flexible with dynamic method calls, especially for methods that might not exist.

---

### 🧲 4. **Use `presence` to handle blank/nil values**

```ruby
record.presence || default_value
```

This returns `record` if it's present, otherwise returns `default_value`.

---

### 🔁 5. **Fallbacks with `||` or `||=`**

```ruby
name = record&.name || "Guest"
```

Or memoization:

```ruby
@user ||= User.find_by(id: session[:user_id])
```

---

### ✅ 6. **Handle `nil` in Controllers (e.g., 404)**

If a record is not found, return a 404:

```ruby
def show
  @post = Post.find_by(slug: params[:slug])
  if @post.nil?
    render file: "#{Rails.root}/public/404.html", status: :not_found
  end
end
```

Or raise an exception:

```ruby
@post = Post.find_by!(slug: params[:slug])
# Raises ActiveRecord::RecordNotFound if not found
```

---

### 🔄 7. **`find_by` vs `find`**

* `find_by` returns `nil` if not found — safe but requires handling.
* `find` raises an exception if not found — useful when you want an automatic 404 via `rescue_from`.

---

### ✨ Summary

| Technique             | Use When                            |     |                                          |
| --------------------- | ----------------------------------- | --- | ---------------------------------------- |
| `if record.nil?`      | Explicit checks                     |     |                                          |
| `record&.name`        | Safe one-liners                     |     |                                          |
| `record.try(:name)`   | Compatibility with older Ruby       |     |                                          |
| \`record.presence     |                                     | x\` | Need fallback only if value is nil/blank |
| \`                    |                                     | =\` | Memoization or default assignment        |
| `find_by` + nil check | Graceful record lookup              |     |                                          |
| `find_by!`            | Raise exception for missing records |     |                                          |

---

Let me know your use case (controller/view/model), and I’ll show you the best way to handle `nil` in that context.
