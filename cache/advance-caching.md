Sweet — let’s level up your caching game in Rails 🔥

If you’re beyond basic `.set/.get` Redis caching and ready for **advanced caching**, here are powerful strategies to consider — tailored for performance, scalability, and developer sanity.

---

## 🚀 1. **Versioned Caching (Key Invalidation Strategy)**

Instead of manually deleting keys on update, use **versioned keys**.

### 🔧 Example:

```ruby
def cache_key_for_product(product)
  "product:#{product.id}:v#{product.updated_at.to_i}"
end
```

```ruby
$redis.set(cache_key_for_product(product), product.to_json)
```

This way:
- No need to manually delete old keys
- Each update writes to a new key
- Keeps cache fresh & avoids race conditions

---

## 🚀 2. **Low-Level Caching with `Rails.cache`**

`Rails.cache` works with Redis and provides automatic key formatting and expiration.

```ruby
Rails.cache.fetch("product:#{product.id}", expires_in: 15.minutes) do
  product.as_json(only: [:id, :title, :price])
end
```

Advantages:
- Cleaner
- Built-in support for Memcached, Redis, FileStore
- Handles race conditions (`race_condition_ttl`)

✅ Works great for fragment caching too!

---

## 🚀 3. **Fragment Caching with View Helpers**

In views:

```erb
<% cache(product) do %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>
```

This caches rendered partials, tied to model `cache_key_with_version`.

---

## 🚀 4. **Collection Caching with Nested Keys**

If you’re caching **lists of items**, cache them by collection key:

```ruby
cache_key = "products:list:category:#{category.id}:v#{category.updated_at.to_i}"

Rails.cache.fetch(cache_key, expires_in: 10.minutes) do
  category.products.includes(:images).as_json(only: [:id, :title])
end
```

✅ This auto-busts cache when category or product is updated.

---

## 🚀 5. **Background Cache Warming (Preload Popular Data)**

Use Sidekiq or ActiveJob to preload data:

```ruby
class CacheWarmProductJob < ApplicationJob
  def perform(product_id)
    product = Product.find(product_id)
    Rails.cache.write("product:#{product.id}", product.as_json, expires_in: 1.hour)
  end
end
```

Trigger on updates or with scheduled jobs.

---

## 🚀 6. **Use Redis Hashes for Grouped Data**

Store related keys under one Redis hash:

```ruby
$redis.hset("category:#{category.id}:products", product.id, product.to_json)
$redis.hget("category:#{category.id}:products", product.id)
```

Benefits:
- Easier to purge all products in a category
- Clean, structured grouping

---

## 🚀 7. **Use Cache Tags with `redis-tags` Gem** (Advanced)

This gem lets you tag cache entries for easier bulk invalidation.

```ruby
Redis::Tags.cache.write("product:1", product_data, tags: ["products", "category:5"])
```

Then:

```ruby
Redis::Tags.expire_tag("category:5")
```

Boom — all related keys are purged 🔥

---

## 🚀 8. **Scheduled Cache Expiry + Prewarm**

Use `sidekiq-cron` or `whenever` gem to periodically refresh popular cache keys:

```ruby
every 10.minutes do
  runner "CacheWarmService.refresh_homepage_products"
end
```

---

## 🚀 9. **Serve Cache-Only API Fallback**

For ultra-speedy API responses:

```ruby
def show
  cache_key = "product:#{params[:id]}"
  if (cached = Rails.cache.read(cache_key))
    render json: JSON.parse(cached) and return
  end

  product = Product.find(params[:id])
  Rails.cache.write(cache_key, product.to_json)
  render json: product
end
```

✅ If DB is slow or down, cached data still serves.

---

## 🧠 Summary: Caching Arsenal

| Feature               | Benefit                               |
|----------------------|----------------------------------------|
| Versioned Keys        | Automatic cache-busting               |
| `Rails.cache`         | Easy key management + expirations     |
| Fragment/Collection Caching | Fast view rendering             |
| Redis Hashes          | Grouped cache & quick invalidation    |
| Background Warming    | Preload frequently accessed data      |
| Tags (`redis-tags`)   | Bulk expiration by category/topic     |
| Scheduled Tasks       | Regular cache refresh                 |

---

Want help implementing one of these?  
Or optimizing for heavy traffic, SSR, or mobile APIs?

Just let me know what you’re building 💬