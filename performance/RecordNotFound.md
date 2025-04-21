Great question — you’re wondering: **what happens if `params[:id]` doesn’t match any product?** Right now, without handling, it throws:

```
ActiveRecord::RecordNotFound
```

This will cause a 500 error unless it's rescued. You want to handle it cleanly and return a **404 Not Found** response instead.

---

### ✅ Solution: Handle `RecordNotFound` Properly

Here are two solid ways to do it:

---

### 💡 Option 1: Rescue in the `set_product` method

```ruby
def set_product
  @product = Product.find(params[:id])
rescue ActiveRecord::RecordNotFound
  render json: { error: "Product not found" }, status: :not_found
end
```

Simple and works well for most cases.

---

### 💡 Option 2: Global rescue in controller (cleaner if reused)

```ruby
class Api::ProductsController < ApplicationController
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found

  before_action :set_product, only: :show

  def index
    products = Product.select(:id, :name, :price).limit(50)
    render json: products, status: :ok
  end

  def show
    render json: @product, status: :ok
  end

  private

  def set_product
    @product = Product.find(params[:id])
  end

  def record_not_found
    render json: { error: 'Product not found' }, status: :not_found
  end
end
```

This way, **any time** `Product.find` fails (even in other actions), you return a proper 404 response, and your code stays clean.

---

Let me know if you want to handle **other errors globally**, like bad params, or want to build a base API controller for reusable error handling.