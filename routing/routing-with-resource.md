To add a **"latest"** URL for the `products` resource while keeping only the `:index` and `:show` actions in your routes, you can simply define a custom route for the latest product, while keeping the existing ones.

Here’s how you can modify your routes to include a `latest` route for `products`:

### Step 1: Modify `config/routes.rb`

You want to add a **custom route** for fetching the latest product alongside the existing `:index` and `:show` actions.

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :products, only: [:index, :show] do
    collection do
      get :latest  # Custom route for the latest product
    end
  end
end
```

### Explanation:

* **`resources :products`**: Keeps the standard routes for `index` and `show`.
* **`collection do`**: This block defines custom routes that apply to the whole collection of `products`.
* **`get :latest`**: Defines the route `/products/latest` to fetch the latest product.

---

### Step 2: Update the `ProductsController`

In your `ProductsController`, define the `latest` action to fetch the most recent product:

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  # GET /products/latest
  def latest
    @latest_product = Product.order(created_at: :desc).first

    if @latest_product
      render json: @latest_product, status: :ok  # Respond with the latest product in JSON format
    else
      render json: { message: 'No products found' }, status: :not_found
    end
  end

  # Other actions like index and show...
  def index
    @products = Product.all
    render json: @products
  end

  def show
    @product = Product.find(params[:id])
    render json: @product
  end
end
```

### Explanation of the `latest` action:

* **`Product.order(created_at: :desc).first`**: This query fetches the latest product by ordering the products based on `created_at` in descending order (most recent first).
* **`render json:`**: If the product exists, it will return the latest product as a JSON response. If no products exist, it returns a "No products found" message with a 404 status.

---

### Step 3: Test the Routes

Now, with the route `/products/latest` defined, you can visit or send an HTTP request to fetch the latest product.

#### Example Request for the Latest Product:

```bash
GET http://localhost:3000/products/latest
```

This will return the latest product in JSON format:

```json
{
  "id": 10,
  "name": "Latest Product",
  "price": 199.99,
  "created_at": "2025-05-01T12:00:00Z",
  "updated_at": "2025-05-01T12:00:00Z"
}
```

---

### Step 4: Optional - Add Caching for Performance

If your `products` table has a large number of records and you're frequently fetching the latest product, you might want to cache the result to reduce database load.

For example, in the `latest` action, you can use `Rails.cache` to cache the result:

```ruby
def latest
  @latest_product = Rails.cache.fetch('latest_product', expires_in: 10.minutes) do
    Product.order(created_at: :desc).first
  end

  if @latest_product
    render json: @latest_product, status: :ok
  else
    render json: { message: 'No products found' }, status: :not_found
  end
end
```

This will store the result of the latest product in the cache for 10 minutes and return the cached value on subsequent requests.

---

### Step 5: Routes Summary

Now, your `config/routes.rb` will look like this:

```ruby
Rails.application.routes.draw do
  resources :products, only: [:index, :show] do
    collection do
      get :latest  # Custom route for fetching the latest product
    end
  end
end
```

This setup allows:

* `GET /products` → To list all products.
* `GET /products/:id` → To show a specific product by ID.
* `GET /products/latest` → To fetch the latest product.

---

### Final Notes:

* You can adjust the controller's `latest` action to render the latest product in a view instead of as JSON if you're working with a web interface instead of an API.
* For large datasets, consider pagination or caching the results to reduce load.

Let me know if you'd like additional customizations or optimizations!
