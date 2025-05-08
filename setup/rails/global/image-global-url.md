To set up a **global external base URL** (like a CDN or media host) in a Rails app, here's the cleanest and most **industry-standard** way to do it — so it's accessible throughout your app (controllers, models, views, helpers, etc.).

---

## ✅ Step-by-Step: Global External Base URL in Rails

---

### 1. **Set the Base URL via ENV**

Create or update your `.env` file (or config it in your server environment):

```env
CDN_BASE_URL=https://cdn.example.com
```

> You'll need the `dotenv-rails` gem for local development:

```ruby
# Gemfile
gem 'dotenv-rails', groups: [:development, :test]
```

---

### 2. **Create a Global App Config (Best Practice)**

Use a Rails initializer for structured configuration.

🗂️ `config/initializers/external_urls.rb`:

```ruby
Rails.application.config.external_urls = {
  cdn_base: ENV['CDN_BASE_URL'] || "https://cdn.example.com"
}
```

---

### 3. **Access It Anywhere in the App**

You can now access it globally:

```ruby
Rails.configuration.external_urls[:cdn_base]
```

#### ✅ Example in Controller or Helper:

```ruby
def external_image_url(path)
  base = Rails.configuration.external_urls[:cdn_base]
  URI.join(base, path).to_s
end
```

---

### 4. **Usage Example**

In a controller or serializer:

```ruby
def product_json(product)
  path = product.thumb_image.presence || "/images/default-product.jpg"
  product.as_json.merge({
    image_url: external_image_url(path)
  })
end
```

---

## ✅ Bonus: Make a Helper for Global Use

Create a helper to make it DRY and accessible everywhere:

🗂️ `app/helpers/url_helper.rb`

```ruby
module UrlHelper
  def external_image_url(path)
    base = Rails.configuration.external_urls[:cdn_base]
    URI.join(base, path).to_s
  end
end
```

And include it in `ApplicationController`:

```ruby
class ApplicationController < ActionController::Base
  include UrlHelper
end
```

---

### ✅ Final Summary

| Step          | Description                                   |
| ------------- | --------------------------------------------- |
| ✅ ENV         | Use `ENV['CDN_BASE_URL']`                     |
| ✅ Initializer | Create `external_urls.rb` to load globally    |
| ✅ Helper      | Use `external_image_url(path)` for clean URLs |
| ✅ Fallback    | Provide a default in case ENV is missing      |

---

Would you like me to scaffold the full helper + initializer for you in one copy-paste-ready file?
