Creating a **helper** in Rails to handle `order_by`, **pagination**, **sorting**, and **searching** is a great way to encapsulate common functionality used across multiple controllers or views — especially in index pages (like for products, users, orders, etc.).

Here's a complete guide with working examples for each functionality:

---

## ✅ 1. **Search, Sort, and Paginate Helper Overview**

We'll build a helper module (can live in a concern, helper, or service) to:

* **Search**: Simple search by name or other fields.
* **Sort**: `order_by` any column with ascending/descending toggle.
* **Paginate**: Use `kaminari` or `pagy` for pagination.

---

## 🔧 Setup

### Add Pagination Gem

#### Option A: **Kaminari**

```ruby
# Gemfile
gem 'kaminari'
```

Then run:

```bash
bundle install
```

#### Option B: **Pagy** (faster/lighter)

```ruby
gem 'pagy'
```

---

## 🧩 Helper Module: `app/helpers/query_helper.rb`

```ruby
# app/helpers/query_helper.rb
module QueryHelper
  def apply_filters(scope, params, searchable_fields: [], sortable_fields: [])
    # Apply search
    if params[:q].present? && searchable_fields.any?
      query = params[:q].downcase
      scope = scope.where(
        searchable_fields
          .map { |f| "LOWER(#{f}) LIKE :q" }
          .join(' OR '),
        q: "%#{query}%"
      )
    end

    # Apply sort
    sort_column = params[:sort].presence_in(sortable_fields) || 'created_at'
    sort_direction = params[:direction] == 'asc' ? 'asc' : 'desc'
    scope = scope.order("#{sort_column} #{sort_direction}")

    # Apply pagination (Kaminari example)
    scope = scope.page(params[:page]).per(params[:per_page] || 20)

    scope
  end
end
```

---

## ✅ Usage in Controller

```ruby
class ProductsController < ApplicationController
  include QueryHelper

  def index
    @products = apply_filters(
      Product.all,
      params,
      searchable_fields: %w[name description],
      sortable_fields: %w[name price created_at]
    )
  end
end
```

---

## 🖼️ View Example (ERB) with Sort Links and Pagination

```erb
<!-- app/views/products/index.html.erb -->
<h1>Products</h1>

<%= form_with url: products_path, method: :get, local: true do %>
  <%= label_tag :q, "Search:" %>
  <%= text_field_tag :q, params[:q] %>
  <%= submit_tag "Search" %>
<% end %>

<table>
  <thead>
    <tr>
      <th><%= sort_link('name') %></th>
      <th><%= sort_link('price') %></th>
      <th><%= sort_link('created_at', 'Created') %></th>
    </tr>
  </thead>
  <tbody>
    <% @products.each do |product| %>
      <tr>
        <td><%= product.name %></td>
        <td><%= product.price %></td>
        <td><%= product.created_at %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<%= paginate @products %>
```

---

## 🧩 Sort Link Helper

Add this to your `ApplicationHelper`:

```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def sort_link(column, title = nil)
    title ||= column.titleize
    direction = (params[:sort] == column && params[:direction] == 'asc') ? 'desc' : 'asc'
    link_to title, params.permit(:q, :page, :per_page).merge(sort: column, direction: direction)
  end
end
```

---

## 🔁 Supports:

| Feature        | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| **Search**     | Basic keyword search on one or more fields (`LIKE` SQL query) |
| **Sort**       | Clickable column headers to sort ascending/descending         |
| **Pagination** | Uses `kaminari` to paginate large results                     |

---

## ✅ Example URL Queries

* `/products?q=laptop` → Search for products with "laptop"
* `/products?sort=price&direction=desc` → Sort by price descending
* `/products?page=2` → Pagination

---

Would you like a version that works with **Pagy** instead of Kaminari or integrates with a specific front-end framework (like Turbo, Bootstrap, Tailwind, etc.)?
