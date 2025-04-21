
Ah nice — you're asking about **logging in Rails**, either for:

- 📦 **Console-level logging** (e.g., while debugging in `rails console`)
- 🌐 **Request-level logging** (e.g., controller logs, SQL, params, etc.)

Here's a full guide to both with some pro tips:

---

## 🔍 1. **Log from the Console (Rails.logger)**

When you're in `rails console`, you can log like this:

```ruby
Rails.logger.info "Something useful"
Rails.logger.debug "Detailed debug info"
Rails.logger.error "Something went wrong"
```

🧠 Tip: It writes to `log/development.log` (or `log/test.log`, etc.)

You can use it inside models, services, or anywhere else in Rails.

---

## 📂 2. **Request-Level Logs (during API or page hit)**

By default, Rails logs:

- HTTP method, path, controller, and action
- Parameters
- SQL queries
- Response status and time

Example from `log/development.log`:
```
Started GET "/api/products" for 127.0.0.1 at 2025-04-21 12:00:00 +0000
Processing by Api::ProductsController#index as JSON
  Product Load (0.2ms)  SELECT * FROM `products`
Completed 200 OK in 5ms
```

You can add **custom logs** in the controller like:

```ruby
def show
  Rails.logger.info "Fetching product with ID #{params[:id]}"
  @product = Product.find(params[:id])
end
```

---

## 🧪 3. **Log Params for Debugging**

Inside a controller:

```ruby
Rails.logger.debug "Params: #{params.inspect}"
```

Or for safer structured logging:

```ruby
Rails.logger.info({ event: "ProductFetch", id: params[:id] }.to_json)
```

---

## 🔧 4. **Adjust Log Level (Optional)**

In `config/environments/development.rb`:

```ruby
config.log_level = :debug  # :info, :warn, :error, :fatal
```

- `:debug` = most verbose (good for dev)
- `:info` = basic flow (good for staging)
- `:warn` and up = minimal logs (good for prod)

---

## 📘 5. **Log to Console in Real-Time (tail)**

Want to watch logs live?

```bash
tail -f log/development.log
```

Or pipe them with grep:

```bash
tail -f log/development.log | grep "Product"
```

---

## 🧪 6. **Log Output in `rails console` Immediately**

By default, logs don’t always print in real time in the console. To fix that:

```ruby
ActiveRecord::Base.logger = Logger.new(STDOUT)
```

Now you’ll see all SQL queries as they happen.

---

Let me know if you want to:
- Send logs to external services (e.g., Logstash, Datadog)
- Colorize or format logs for dev
- Add logging in background jobs, services, or initializers

Happy logging 🚀