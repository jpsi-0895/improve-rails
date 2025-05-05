To analyze **incoming requests** and log **response times** in a Rails app, you can configure Rails or add middleware to log detailed request metrics, including:

- HTTP method and path
- Parameters
- Status codes
- Response times
- Timestamps
- IP address or user agent (if needed)

---

## ✅ 1. **Lograge for Clean Request/Response Logs (Production-Ready)**

### 🔧 Step 1: Add to Gemfile

```ruby
gem 'lograge'
```

Then run:

```bash
bundle install
```

---

### 🔧 Step 2: Enable in `config/environments/production.rb`:

```ruby
config.lograge.enabled = true

# Log as key-value pairs
config.lograge.formatter = Lograge::Formatters::KeyValue.new

# Add custom data (e.g., response time, params)
config.lograge.custom_options = lambda do |event|
  {
    time: Time.now.utc.iso8601,
    duration: event.duration.round(2),
    status: event.payload[:status],
    params: event.payload[:params].except("controller", "action"),
    ip: event.payload[:remote_ip]
  }
end
```

Optionally log to JSON:

```ruby
config.lograge.formatter = Lograge::Formatters::Json.new
```

---

### 📝 Example Output (Key-Value Format):

```
method=GET path=/api/products status=200 duration=42.3 params={"q"=>"shoes"} time=2025-05-02T14:00:00Z
```

---

## 🧰 2. **Manual Middleware (If You Don't Want Lograge)**

Create a custom middleware to log request + response times:

```ruby
# lib/middleware/request_timer.rb
class RequestTimer
  def initialize(app)
    @app = app
  end

  def call(env)
    start_time = Time.now
    status, headers, response = @app.call(env)
    duration = ((Time.now - start_time) * 1000).round(2)

    request = Rack::Request.new(env)
    Rails.logger.info "Request: #{request.request_method} #{request.fullpath} | Status: #{status} | Duration: #{duration}ms"

    [status, headers, response]
  end
end
```

Register it in `config/application.rb`:

```ruby
require_relative '../lib/middleware/request_timer'
config.middleware.use RequestTimer
```

---

## 🔍 3. **Rails’ Built-in ActionController Logging**

Rails logs include response times by default in development, e.g.:

```
Processing by ProductsController#index as JSON
  Product Load (1.2ms) ...
Completed 200 OK in 25ms (Views: 3.4ms | ActiveRecord: 10.2ms)
```

You can extract these in logs by filtering on `Completed` lines.

---

## 📊 4. **Monitor with External Tools**

You can ship these logs to tools like:

- **ELK (Elasticsearch + Logstash + Kibana)**
- **Grafana Loki**
- **Datadog / LogDNA / Papertrail**
- **Sentry or New Relic** (with performance monitoring)

They let you visualize:
- Slow endpoints
- Status code distribution
- API errors over time

---

## ✅ Summary

| Goal | Best Tool |
|------|-----------|
| Clean logs with durations | `lograge` |
| Full control | Custom middleware |
| Error & perf alerts | Sentry, New Relic |
| Real-time search | ELK, Loki, Datadog |

---

Would you like a full example of sending structured logs to something like **Logstash** or **Grafana Loki**?