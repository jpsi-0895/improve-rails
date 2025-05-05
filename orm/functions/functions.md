To **get a product by slug** and **fetch only 5 products** (e.g. related, recent, or featured) in Rails, here’s a clean and effective way to do it.

---

## ✅ Goal:

* Fetch a product by its **slug** (instead of ID).
* Fetch **5 other products** (you can define the criteria: latest, same category, etc.).

---

## 📦 1. Product Model with Slug

Ensure your `Product` model has a `slug` column:

```bash
rails generate migration AddSlugToProducts slug:string:uniq
```

In the model:

```ruby
# app/models/product.rb
class Product < ApplicationRecord
  validates :slug, presence: true, uniqueness: true

  def to_param
    slug
  end
end
```

---

## 🔍 2. Fetch Product by Slug in Controller

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  def show
    @product = Product.find_by!(slug: params[:id])
    @related_products = Product.where.not(id: @product.id).limit(5)

    render json: {
      product: @product,
      related_products: @related_products
    }
  end
end
```

**Note**: `params[:id]` works because of `to_param` method overriding `product_path(@product)` to use slug.

---

## 🧪 Example Routes

```ruby
# config/routes.rb
resources :products, only: [:show]
```

So now visiting `/products/my-product-slug` will hit `ProductsController#show` and load the product by slug.

---

## 📝 Optional Improvements

* **Scope for related products**:

  ```ruby
  scope :related_to, ->(product) {
    where(category_id: product.category_id).where.not(id: product.id).limit(5)
  }
  ```

  Usage:

  ```ruby
  @related_products = Product.related_to(@product)
  ```

* **Cache product by slug with Redis** if performance is critical.

---

Would you like to add Redis caching for this product lookup or paginate the product list instead of a hard limit?
