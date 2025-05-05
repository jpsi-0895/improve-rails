Great! Here's a complete working example of how to use **Pagy** in a Rails application for efficient, fast pagination — including setup, controller, view, and helper code.

---

## ✅ 1. **Install Pagy**

Add the Pagy gem to your `Gemfile`:

```ruby
# Gemfile
gem 'pagy'
```

Then install it:

```bash
bundle install
```

---

## ✅ 2. **Pagy Setup**

Run the Pagy installer:

```bash
bin/rails generate pagy:install
```

This will create `config/initializers/pagy.rb`, where you can customize default settings.

---

## ✅ 3. **Include Pagy in Controller**

In any controller where you want pagination:

```ruby
# app/controllers/products_controller.rb
class ProductsController < ApplicationController
  include Pagy::Backend

  def index
    @pagy, @products = pagy(Product.order(created_at: :desc))
  end
end
```

---

## ✅ 4. **Render Pagy Pagination in Views**

In your view file (`app/views/products/index.html.erb`):

```erb
<h1>Products</h1>

<ul>
  <% @products.each do |product| %>
    <li><%= product.name %> - $<%= product.price %></li>
  <% end %>
</ul>

<%= pagy_nav(@pagy) %>
```

---

## ✅ 5. **Include Pagy in ApplicationHelper**

Pagy uses its own view helpers — include them in your application helper to make them available in all views:

```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  include Pagy::Frontend
end
```

---

## ✅ 6. **Optional: Bootstrap/Tailwind Integration**

If you're using a CSS framework like Bootstrap, install the relevant frontend components:

```bash
bin/rails generate pagy:extras
```

Then enable the required module in `config/initializers/pagy.rb`, e.g., for Bootstrap:

```ruby
require 'pagy/extras/bootstrap'
```

Then update your helper and view:

```erb
<%= pagy_bootstrap_nav(@pagy) %>
```

---

## ✅ 7. **Customize Per-Page Count**

To change how many items are shown per page:

```ruby
@pagy, @products = pagy(Product.all, items: 10)
```

Or allow user control:

```ruby
@pagy, @products = pagy(Product.all, items: params[:per_page] || 20)
```

---

## 🔄 Full Example

```ruby
# products_controller.rb
def index
  @pagy, @products = pagy(
    Product.where('name ILIKE ?', "%#{params[:q]}%").order(created_at: :desc),
    items: params[:per_page] || 10
  )
end
```

```erb
<!-- index.html.erb -->
<%= form_with url: products_path, method: :get, local: true do %>
  <%= label_tag :q, "Search" %>
  <%= text_field_tag :q, params[:q] %>
  <%= submit_tag "Go" %>
<% end %>

<% @products.each do |product| %>
  <p><%= product.name %></p>
<% end %>

<%= pagy_nav(@pagy) %>
```

---

## 📌 Why Use Pagy?

* ⚡ **Much faster** than Kaminari/WillPaginate
* 🧼 Minimal and easy to customize
* 🔄 Great integration with Tailwind/Bootstrap

---

Would you like this example extended with sorting or a reusable helper for filtering and paginating resources?
