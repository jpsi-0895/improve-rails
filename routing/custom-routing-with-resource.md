To achieve a **custom URL** for the latest product while keeping the action name as `latest`, you can explicitly specify the route path in `config/routes.rb`. This will allow you to control both the URL and the corresponding controller action.

For example, if you want the URL to be something like `/latest-product` instead of `/products/latest`, here's how you can set it up.

### Step 1: Modify `config/routes.rb` to define a custom path

In your `config/routes.rb`, you can define a custom URL path for the `latest` action by specifying a custom route:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :products, only: [:index, :show]

  # Custom route for the latest product with a custom URL
  get 'latest-product', to: 'products#latest', as: :latest_product
end
```

### Explanation:

* **`get 'latest-product'`**: This sets the custom URL path to `/latest-product`.
* **`to: 'products#latest'`**: This maps the custom URL to the `latest` action in the `ProductsController`.
* **`as: :latest_product`**: This gives a named route `latest_product_path` (useful in views and redirects).

---

### Step 2: Define the `latest` Action in the `ProductsController`

Now, in your `ProductsController`, define the `latest` action as before:

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  # GET /latest-product
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

### Explanation:

* **`Product.order(created_at: :desc).first`**: This queries the latest product by `created_at` in descending order and returns the most recent product.
* **Rendering as JSON**: We're returning the result as JSON. You can change this to render a view if you're working with HTML.

---

### Step 3: Test the Custom URL

Now, the custom URL will be `/latest-product`, and when you visit that URL, it will call the `latest` action in the `ProductsController`.

#### Example Request:

```bash
GET http://localhost:3000/latest-product
```

This will return the latest product in JSON format, for example:

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

### Step 4: Using the Named Route (`latest_product_path`)

In your views or redirects, you can use the named route `latest_product_path`:

#### Example in a View:

```erb
<%= link_to 'Latest Product', latest_product_path %>
```

This generates a link to the `/latest-product` URL.

#### Example in a Controller (Redirect):

```ruby
redirect_to latest_product_path
```

This will redirect to `/latest-product`.

---

### Final `config/routes.rb`:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :products, only: [:index, :show]

  # Custom route for the latest product with a custom URL
  get 'latest-product', to: 'products#latest', as: :latest_product
end
```

### Recap:

* **Custom URL**: `/latest-product` (instead of `/products/latest`).
* **Controller action**: `latest` (the action name doesn't change).
* **Named route**: `latest_product_path` for linking or redirects.

This setup gives you full control over the URL and action mapping in Rails. Let me know if you need further adjustments or clarifications!
