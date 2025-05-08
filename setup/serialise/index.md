To improve your `ProductsController` and make it follow **SOLID principles** (especially **Single Responsibility** and **Open/Closed Principle**), we can:

---

## ✅ Goals:

1. Move formatting logic (like `product_json`) into a **Presenter** or **Serializer**.
2. Separate **caching** and **data access** from controller logic.
3. Simplify controller responsibilities: it should handle **request/response**, not data shaping or caching.
4. Ensure **consistency** in `index`, `show`, and `latest`.

---

## ✨ Refactored Code — Clean & SOLID

### ✅ `app/controllers/api/products_controller.rb`

```ruby
class Api::ProductsController < ApplicationController
  before_action :set_product, only: [:show]

  def index
    products = ProductCacheService.new.fetch_products
    render json: products.map { |product| ProductSerializer.new(product) }, status: :ok
  end

  def show
    render json: ProductSerializer.new(@product), status: :ok
  end

  def latest
    product = ProductCacheService.new.fetch_latest_product

    if product
      render json: ProductSerializer.new(product), status: :ok
    else
      render json: { message: 'No products found' }, status: :not_found
    end
  end

  private

  def set_product
    @product = Product.find_by(id: params[:id])

    return if @product.present?

    render json: { message: 'Product not found' }, status: :not_found
  end
end
```

---

### ✅ `app/serializers/product_serializer.rb`

```ruby
class ProductSerializer
  def initialize(product)
    @product = product
  end

  def as_json(*)
    {
      id: @product[:id],
      title: @product[:title],
      field: @product[:field],
      type: @product[:type],
      image_url: full_image_url(@product[:thumb_image])
    }
  end

  private

  def full_image_url(path)
    return nil if path.blank?

    base = Rails.configuration.external_urls[:cdn_base]
    URI.join(base, path).to_s
  end
end
```

---

### ✅ `config/initializers/external_urls.rb`

```ruby
Rails.application.config.external_urls = {
  cdn_base: ENV['CDN_BASE_URL'] || 'https://cdn.example.com'
}
```

---

## 🔍 Why This is SOLID

| Principle                 | How It’s Applied                                                             |
| ------------------------- | ---------------------------------------------------------------------------- |
| **S**ingle Responsibility | Controller only handles HTTP logic. Serializer handles data formatting.      |
| **O**pen/Closed           | You can add more fields to `ProductSerializer` without changing controller.  |
| **L**iskov                | You can safely replace `Product` with a decorated version or cached version. |
| **I**nterface Segregation | Small, single-purpose methods and services.                                  |
| **D**ependency Inversion  | Uses initializer config + DI for base URL instead of hardcoding.             |

---

Would you like a version of this with ActiveModel::Serializers or FastJsonapi for even cleaner JSON API support?
