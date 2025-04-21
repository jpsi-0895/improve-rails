Gotcha — you want a **Rails API-only app**, *no Stimulus, no Turbo, no frontend/view stuff at all*. Just a clean backend Rails API for serving JSON. Here's how to set that up:

---

### ✅ Step-by-Step: Create a Rails API-only App (No View Layer)

#### 1. **Create the API-Only App**
Run this to generate a Rails app with just API functionality:

```bash
rails new my_api_app --api -d mysql
cd my_api_app
```

The `--api` flag tells Rails to skip views, helpers, assets, and frontend JS stuff like Turbo or Stimulus. It also configures Rails for JSON-only responses.

---

#### 2. **Set Up MySQL Connection**

Edit your `config/database.yml` to connect to your existing database:

```yaml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: 5
  username: your_mysql_user
  password: your_mysql_password
  host: localhost

development:
  <<: *default
  database: your_existing_database_name
```

Then install dependencies:

```bash
bundle add mysql2
bundle install
```

---

#### 3. **Generate Models Without Migrations (use existing tables)**

For an existing table, like `customers`:

```bash
rails generate model Customer --skip-migration
```

Rails will infer columns from the database when you use ActiveRecord.

---

#### 4. **Create a Simple API Controller**

```bash
rails generate controller api/customers --skip-template-engine --no-helper --no-assets
```

Then edit the controller to return JSON:

```ruby
# app/controllers/api/customers_controller.rb
module Api
  class CustomersController < ApplicationController
    def index
      customers = Customer.all
      render json: customers
    end

    def show
      customer = Customer.find(params[:id])
      render json: customer
    end
  end
end
```

---

#### 5. **Configure Routes**

In `config/routes.rb`:

```ruby
Rails.application.routes.draw do
  namespace :api do
    resources :customers, only: [:index, :show]
  end
end
```

---

#### 6. **Test it out!**

Start your server:

```bash
rails server
```

Then hit:

- `GET http://localhost:3000/api/customers`
- `GET http://localhost:3000/api/customers/1`

You’ll get JSON back.

---

Let me know if you want to add auth (like JWT), version your API, or hook it to a frontend (like React, Vue, etc). Want help auto-generating models from existing MySQL tables too?