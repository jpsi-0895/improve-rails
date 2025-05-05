Setting up **logging and monitoring logs** in a **Rails application** is critical for diagnosing issues, auditing activity, and understanding app behavior—especially in production.

---

## ✅ 1. **Default Logging in Rails**

Rails uses the `ActiveSupport::Logger` class.

### 📁 Default Log Files
- `log/development.log`
- `log/production.log`
- `log/test.log`

---

## 🔧 2. **Configure Logging Level**

In `config/environments/production.rb` or `development.rb`:

```ruby
config.log_level = :info  # options: :debug, :info, :warn, :error, :fatal, :unknown
```

- Use `:debug` in development for verbose logs.
- Use `:info` or `:warn` in production for performance and clarity.

---

## 🧰 3. **Customize Log Format**

In `config/application.rb` or an initializer:

```ruby
config.log_formatter = ::Logger::Formatter.new
```

Or define a custom formatter:

```ruby
class CustomLoggerFormatter < Logger::Formatter
  def call(severity, time, progname, msg)
    "#{time.utc.iso8601} #{severity}: #{msg}\n"
  end
end

config.logger = Logger.new(STDOUT)
config.logger.formatter = CustomLoggerFormatter.new
```

---

## 📦 4. **Use Structured Logging (Optional)**

For JSON logs (useful in Docker or log aggregation systems like ELK, Loki, or CloudWatch):

```ruby
gem 'lograge'
```

Then in `config/environments/production.rb`:

```ruby
config.lograge.enabled = true
config.lograge.formatter = Lograge::Formatters::Json.new
```

---

## 🔍 5. **Monitoring Logs Locally**

You can monitor logs in real-time using:

```bash
tail -f log/production.log
```

Or with more context:

```bash
tail -n 100 -f log/production.log | grep "ERROR"
```

---

## 🚀 6. **Log Monitoring Tools**

For large apps or production environments, use centralized logging tools:

| Tool           | Setup                                            |
|----------------|--------------------------------------------------|
| **ELK Stack** (Elasticsearch, Logstash, Kibana) | Sends logs to Logstash, visualized in Kibana |
| **Loki + Grafana**            | Lightweight, great for metrics and logs |
| **AWS CloudWatch Logs**       | Managed log ingestion for EC2/ECS |
| **LogDNA / Datadog / Papertrail** | SaaS log aggregators with alerts |

### Example: Sending Rails logs to Syslog

```ruby
config.logger = Syslog::Logger.new 'my_app'
```

---

## 📊 7. **Set Up Alerts**

With tools like:
- **Prometheus + Grafana**
- **Sentry** (for error monitoring)
- **Datadog** or **New Relic** (for logs and performance)

You can alert on:
- Too many 500s
- Long response times
- Specific error patterns

---

## 🧪 8. **Test Logging**

From anywhere in your app:

```ruby
Rails.logger.info "User signed in: #{user.id}"
Rails.logger.error "Something went wrong: #{e.message}"
```

---

## ✅ Summary

| Task | Tool / Config |
|------|----------------|
| Set log level | `config.log_level = :info` |
| Real-time logs | `tail -f log/production.log` |
| Structured logs | Use `lograge` or JSON formatter |
| Centralized logs | ELK, Loki, CloudWatch, etc. |
| Monitoring & alerts | Datadog, Grafana, Sentry |

---

Would you like a ready-to-use logging setup for production with **log aggregation** or **JSON logs** for Docker?