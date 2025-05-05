If you want to use **Redis** for caching a **single record** in **Rails** without relying on `Rails.cache`, you can directly use the **Redis** client to set and get values. Below is a step-by-step guide on how to set up Redis for caching a single record (such as a product) and manually handle the cache without the built-in Rails cache store.

### Step 1: Install Redis and the Redis Client Gem

Ensure you have the `redis` gem in your `Gemfile`:

```ruby
gem 'redis'
```

Then, run `bundle install`:

```bash
bundle install
```

### Step 2: Set Up the Redis Client

In Rails, you can configure and initialize Redis directly by creating a custom Redis client in an initializer.

Create an initializer for Redis in `config/initializers/redis.rb`:

```ruby
# config/initializers/redis.rb
$redis = Redis.new(url: ENV['REDIS_URL'] || 'redis://localhost:6379/1')
```

Here, the Redis client is set up globally using the `$redis` variable. This makes Redis available for direct access anywhere in your application.

### Step 3: Create a Service for Caching a Single Record

Create a service class that interacts directly with Redis to cache and fetch a single record (like a **Product**). We will manually handle Redis operations.

#### Example: Caching and Fetching Product by ID

1. **Create the `RecordCacheService` class**:

```ruby
# app/services/record_cache_service.rb
class RecordCacheService
  def initialize(model_class, id)
    @model_class = model_class
    @id = id
    @cache_key = "#{model_class.name.downcase}_#{id}"
  end

  # Fetch the record from Redis or database
  def fetch
    cached_record = get_from_cache
    return cached_record if cached_record

    # If not cached, fetch from DB and cache it
    record = fetch_from_db
    cache(record) if record
    record
  end

  private

  # Get the record from Redis cache
  def get_from_cache
    cached_data = $redis.get(@cache_key)
    return nil unless cached_data

    # Deserialize the cached data (assuming it's stored as JSON)
    JSON.parse(cached_data, object_class: @model_class)
  end

  # Fetch the record from the database
  def fetch_from_db
    @model_class.find_by(id: @id)
  end

  # Cache the record in Redis
  def cache(record)
    # Serialize the record into JSON before caching
    $redis.setex(@cache_key, 10.minutes.to_i, record.to_json)
  end
end
```

### Explanation:

* **`@model_class`** and **`@id`** are passed to the service to identify the specific record (for example, a `Product` model and its `id`).
* **`get_from_cache`**: Attempts to fetch the record from Redis. If the data is found in Redis, it returns it after deserializing (using `JSON.parse`).
* **`fetch_from_db`**: Fetches the record from the database if it's not cached.
* **`cache`**: Caches the fetched record in Redis with an expiration time of 10 minutes (`setex`).

### Step 4: Use the Service in the Controller

Now, in your controller, you can use this service to handle caching logic for a **single product**.

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  def show
    # Use the cache service to fetch the product
    cache_service = RecordCacheService.new(Product, params[:id])
    @product = cache_service.fetch

    if @product
      render json: @product, status: :ok
    else
      render json: { message: 'Product not found' }, status: :not_found
    end
  end
end
```

### Step 5: Clear the Cache (Optional)

If you want to clear the cache when the product is updated or deleted, you can use **ActiveRecord callbacks** to invalidate the cache.

```ruby
# app/models/product.rb
class Product < ApplicationRecord
  after_save :clear_cache
  after_destroy :clear_cache

  private

  def clear_cache
    $redis.del("product_#{self.id}")
  end
end
```

### Explanation:

* Whenever a product is updated or deleted, the cache is cleared by calling `$redis.del("product_#{self.id}")`.
* This ensures that the cache stays up-to-date when the product data changes.

### Step 6: Testing the Cache

1. When you first visit `/products/:id`, the cache is **missed** (i.e., no cached data in Redis). The record will be fetched from the database, cached, and returned to the client.
2. On subsequent visits to `/products/:id`, Redis will **hit the cache**, returning the cached product directly without querying the database.

### Example Redis Commands

If you want to check if the product has been cached, you can use `redis-cli`:

```bash
redis-cli get product_1
```

This will show you the cached value of the product with ID `1`. The cached data is stored in **JSON format**, so you’ll likely see a string representation of the serialized product.

---

### Conclusion

By following these steps, you can directly use **Redis** to cache a **single record** (like a **Product**) in Rails, without relying on `Rails.cache`. The **`RecordCacheService`** provides a flexible, reusable solution for caching, fetching, and invalidating single records manually.

This approach:

* Uses **Redis** directly via the Redis client (`$redis`).
* Stores the serialized data as **JSON** for easy retrieval.
* Ensures that the cache is updated when the underlying record is modified.

Let me know if you need any further clarification or if you'd like additional features added to this solution!
