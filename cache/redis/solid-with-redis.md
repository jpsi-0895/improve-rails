Your current implementation is close, but you're looking for a more **dynamic** cache solution with better structure that adheres to the **SOLID** principles.

The SOLID principles guide object-oriented design in a way that makes it more maintainable, flexible, and easier to test. Specifically, for **Single Responsibility** and **Open/Closed** principles, we can refactor your service class and split responsibilities.

### Key Improvements

1. **Dynamic cache key**: The `cache_key` should dynamically adapt based on parameters, e.g., `category`, `limit`, `offset`.
2. **Separation of concerns**: Handling Redis interactions separately from the caching logic.
3. **Adhering to SOLID principles**: We will extract caching logic into a separate class to ensure that our classes have a single responsibility.

### Refactor to Follow SOLID Principles

We’ll break the service down into smaller classes to ensure the separation of concerns. The three main classes will be:

* `ProductCacheService`: Handles fetching products.
* `RedisCache`: Encapsulates the logic for interacting with Redis.
* `CacheKeyGenerator`: Handles the dynamic generation of cache keys.

### Step 1: Create the Redis Cache Class

This class will encapsulate all the Redis interactions.

```ruby
# app/services/redis_cache.rb
class RedisCache
  def initialize(redis = $redis)
    @redis = redis
  end

  def get(key)
    cached = @redis.get(key)
    JSON.parse(cached) if cached.present?
  end

  def set(key, value, expiry = 10.minutes)
    @redis.setex(key, expiry.to_i, value.to_json)
  end

  def delete(key)
    @redis.del(key)
  end
end
```

### Explanation:

* **`get`**: Retrieves and deserializes the cached data.
* **`set`**: Sets a cache with a specific expiry time. The value is serialized to JSON.
* **`delete`**: Deletes a specific cache key.

### Step 2: Create the Cache Key Generator

This class is responsible for generating the cache key dynamically based on filter parameters.

```ruby
# app/services/cache_key_generator.rb
class CacheKeyGenerator
  def initialize(base_key = 'products:all')
    @base_key = base_key
  end

  def generate(params = {})
    filters = params.to_param
    "#{@base_key}:#{Digest::MD5.hexdigest(filters)}"
  end
end
```

### Explanation:

* **`generate`**: Generates a dynamic cache key based on the filter parameters (`params`). The filters are converted into a query string using `to_param` and hashed with `MD5` to ensure uniqueness.

### Step 3: Refactor Product Cache Service

Now we refactor the `ProductCacheService` to use `RedisCache` and `CacheKeyGenerator`. We also follow the **Single Responsibility Principle** (SRP) by delegating the caching responsibilities to the appropriate classes.

```ruby
# app/services/product_cache_service.rb
class ProductCacheService
  CACHE_KEY = 'products:all'.freeze
  LATEST_CACHE_KEY = 'products:latest'.freeze
  EXPIRY = 10.minutes

  def initialize(redis_cache = RedisCache.new, cache_key_generator = CacheKeyGenerator.new)
    @redis_cache = redis_cache
    @cache_key_generator = cache_key_generator
  end

  # Fetch all products with optional filters and pagination
  def fetch_products(params = {})
    cache_key = @cache_key_generator.generate(params)
    cached = @redis_cache.get(cache_key)
    return cached if cached.present?

    products = fetch_products_from_db(params)
    @redis_cache.set(cache_key, products, EXPIRY)
    products
  end

  # Fetch the latest product
  def fetch_latest_product
    cached = @redis_cache.get(LATEST_CACHE_KEY)
    return cached if cached.present?

    product = fetch_latest_product_from_db
    @redis_cache.set(LATEST_CACHE_KEY, product, EXPIRY)
    product
  end

  # Clear all caches
  def clear_cache
    @redis_cache.delete(CACHE_KEY)
    @redis_cache.delete(LATEST_CACHE_KEY)
  end

  private

  def fetch_products_from_db(params)
    Product.all
           .select(:id, :title, :thumb_image, :field, :type)
           .where(filter_conditions(params))
           .limit(params[:limit] || 20)
           .offset(params[:offset] || 0)
           .as_json
  end

  def fetch_latest_product_from_db
    Product.latest.first&.as_json
  end

  def filter_conditions(params)
    conditions = {}
    conditions[:category] = params[:category] if params[:category].present?
    conditions[:type] = params[:type] if params[:type].present?
    # Add more filters as needed
    conditions
  end
end
```

### Explanation:

* **Initialization**: The class now takes dependencies (`RedisCache` and `CacheKeyGenerator`) via dependency injection. This makes it more testable and adheres to the **Open/Closed Principle** because we can extend the cache logic or cache key generation without modifying the `ProductCacheService` class itself.
* **`fetch_products`**: Fetches products, either from Redis or the database, and caches them.
* **`fetch_latest_product`**: Retrieves the latest product, either from Redis or the database.
* **`clear_cache`**: Clears the cache for both `products:all` and `products:latest`.

### Step 4: Example Usage

Now, you can use the `ProductCacheService` in your controller like so:

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  def index
    cache_service = ProductCacheService.new
    @products = cache_service.fetch_products(params)
    render json: @products
  end

  def show
    cache_service = ProductCacheService.new
    @product = cache_service.fetch_latest_product
    render json: @product
  end

  def clear_cache
    cache_service = ProductCacheService.new
    cache_service.clear_cache
    render json: { message: 'Cache cleared' }
  end
end
```

### Step 5: Testable, Flexible, and Scalable

With this refactor:

* **Testability**: Each class has a single responsibility, making it easier to write unit tests.
* **Extensibility**: You can now easily add more cache-related logic or filters without changing the `ProductCacheService`. For instance, you could implement a new cache layer for categories.
* **Separation of Concerns**: Redis interactions, cache key generation, and product fetching are all separated into their own classes, adhering to SOLID principles.

### Conclusion

This refactor adheres to **SOLID principles**, particularly the **Single Responsibility Principle (SRP)** and the **Open/Closed Principle (OCP)**. You’ve separated caching logic, Redis interactions, and cache key generation, making your code easier to maintain, extend, and test.

Let me know if you need further clarification or additional improvements!
