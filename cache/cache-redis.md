Great idea—caching product data with **Redis** in Rails can **drastically improve performance**, especially for frequently accessed or read-heavy endpoints like product listings or details.

Here’s a complete guide on how to **cache product data using Redis** in a Rails app, the right way.

---

## 🧰 Caching Product Data in Redis with Rails

### ✅ Use case: Speed up read access to product data (e.g. `/api/products/:id`)

---

### 📦 1. Install Redis

**Add Redis gem** to your app:

```ruby
# Gemfile
gem 'redis'
```

```bash
bundle install
```

Ensure Redis is running locally or in your cloud environment (e.g. `localhost:6379` or Redis Cloud, Upstash, etc.)

---

### ⚙️ 2. Configure Redis

In `config/initializers/redis.rb`:

```ruby
$redis = Redis.new(url: ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" })
```

Add `REDIS_URL` to `.env` if needed.

---

### 🧠 3. Cache Product Data

Let’s say you have this controller:

```ruby
class Api::ProductsController < ApplicationController
  def show
    product = fetch_product(params[:id])
    render json: product
  end

  private

  def fetch_product(id)
    key = "product:#{id}"

    cached = $redis.get(key)
    return JSON.parse(cached) if cached

    product = Product
                .includes(:category)
                .select(:id, :title, :price, :category_id)
                .find_by(id: id)

    return {} unless product

    data = {
      id: product.id,
      title: product.title,
      price: product.price,
      category: {
        id: product.category.id,
        title: product.category.title
      }
    }

    $redis.set(key, data.to_json, ex: 10.minutes.to_i) # expires in 10 mins

    data
  end
end
```

---

### 🔄 4. Invalidate Cache on Update/Delete

Whenever product data is updated or deleted, expire the cache:

```ruby
# In Product model or controller

after_save :expire_cache
after_destroy :expire_cache

def expire_cache
  $redis.del("product:#{id}")
end
```

OR if you want a global cache invalidation approach, use a `cache_version` or timestamp (let me know and I’ll show that too).

---

### 🧪 5. Test It

Make two requests to `/api/products/1` and watch the **second one be served from Redis** instantly.

To inspect Redis keys:

```bash
redis-cli
keys product:*
get product:1
```

---

### 🔐 Bonus Tips

- Use **namespaces** like `appname:env:product:id` to prevent key collisions
- Store **collections** (like all products in a category) as `product:category:5`
- Use JSON serialization and compression (e.g. `Oj`, `Zlib`) for large payloads
- You can also use **Rails built-in cache store** with Redis backend

---

### 🧰 Alternate: Rails Cache API with Redis Store

You can also use:

```ruby
Rails.cache.fetch("product:#{id}", expires_in: 10.minutes) do
  expensive_query
end
```

Configure in `config/environments/production.rb`:

```ruby
config.cache_store = :redis_cache_store, {
  url: ENV["REDIS_URL"],
  namespace: "myapp_cache"
}
```

---

Let me know if you want to **cache collections**, use **Rails.cache** instead of direct Redis, or integrate this with a **background job to warm cache**!