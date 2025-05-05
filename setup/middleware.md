In **Rails**, **middleware** refers to a stack of components that process an HTTP request **before it reaches your Rails controller** and **after your controller returns a response**. Middleware can modify the request/response, handle authentication, logging, compression, and more.

---

## 🧱 What Is Middleware?

Think of middleware as a **pipeline**: each middleware component has a chance to process the request and response.

### Request flow:
```
Client → Middleware 1 → Middleware 2 → ... → Rails App
```

### Response flow:
```
Rails App → Middleware N → ... → Middleware 1 → Client
```

---

## 📋 Viewing the Middleware Stack

You can see your app's middleware stack with:

```bash
rails middleware
```

Example output:
```
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActionDispatch::RequestId
...
run MyApp::Application.routes
```

---

## 🔧 Common Built-in Middleware

| Middleware                     | Purpose |
|--------------------------------|---------|
| `Rack::Runtime`               | Adds `X-Runtime` header |
| `ActionDispatch::Static`     | Serves static files |
| `ActionDispatch::Session::CookieStore` | Handles session cookies |
| `Rack::MethodOverride`       | Enables `_method` param for PUT/PATCH/DELETE |
| `Rack::Attack` (optional)    | Rate limiting, throttling |
| `Warden::Manager` (used by Devise) | Auth framework |

---

## ➕ Adding Custom Middleware

You can write and insert your own middleware to manipulate requests or responses.

### Example: Custom Logger Middleware

```ruby
# lib/middleware/request_logger.rb
class RequestLogger
  def initialize(app)
    @app = app
  end

  def call(env)
    Rails.logger.info "Request URL: #{env['REQUEST_URI']}"
    @app.call(env)  # Continue the chain
  end
end
```

### Register the Middleware

In `config/application.rb`:

```ruby
require_relative '../lib/middleware/request_logger'

module YourApp
  class Application < Rails::Application
    config.middleware.use RequestLogger
  end
end
```

---

## 🔁 Reordering or Removing Middleware

You can **insert**, **replace**, or **remove** middleware in `config/application.rb` or an initializer:

```ruby
# Insert before a specific middleware
config.middleware.insert_before Rack::Runtime, MyCustomMiddleware

# Insert after a specific middleware
config.middleware.insert_after ActionDispatch::Static, MyCustomMiddleware

# Remove a middleware
config.middleware.delete Rack::Sendfile
```

---

## 🧰 When to Use Middleware

Use middleware when:
- You need to **intercept all requests/responses**
- You want to enforce **cross-cutting concerns** (e.g. logging, auth, rate limiting)
- You don't need access to **controllers, params, or views**

> For app-specific logic (like resource-based auth or business rules), use **controllers or service objects** instead.

---

## 🧪 Middleware Debugging Tip

You can temporarily log the middleware call chain by modifying a middleware's `call` method and printing `env` or using `Rails.logger`.

---

Would you like an example of middleware for a specific task like rate limiting, CORS, or logging headers?