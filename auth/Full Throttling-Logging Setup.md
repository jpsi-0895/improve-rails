Awesome! Let’s level up your **API security** with **throttling** and **logging** for your **API key authentication** system.

With **throttling**, we can limit the number of requests a client can make to your API over a period of time (to avoid abuse). And with **logging**, we can track all API access attempts (successful and unsuccessful), which will help with debugging and security audits.

Here’s how you can implement both in your Rails API:

---

### 🔒 **Step-by-Step: Full Throttling + Logging Setup**

---

### 1. **Throttling Requests with `rack-attack`**

First, let’s add **rate limiting (throttling)** using the `rack-attack` gem.

#### 1.1. **Add `rack-attack` to Your Gemfile**

```ruby
gem 'rack-attack'
```

Then run:

```bash
bundle install
```

---

#### 1.2. **Set Up Rack::Attack Configuration**

Create a new initializer for `rack-attack`:

```bash
# config/initializers/rack_attack.rb
class Rack::Attack
  # Throttle requests by IP (or API key if preferred)
  throttle('api/ip', limit: 100, period: 1.hour) do |req|
    req.ip
  end

  # Throttle requests by API key (you can also combine this with IP)
  throttle('api/api_key', limit: 500, period: 1.hour) do |req|
    req.headers['X-API-KEY']
  end

  # Block requests if too many attempts are made within a short window
  blocklist('fail2ban') do |req|
    # Block IPs making too many invalid API requests
    req.ip if req.path.start_with?('/api/') && req.headers['X-API-KEY'].nil?
  end

  # Allow all traffic to /health, /status, or other non-API endpoints
  safelist('allow health check') do |req|
    req.path == '/api/health' || req.path == '/api/status'
  end
end
```

In this setup:

- **Throttling by IP:** Limit the number of requests from the same IP to 100 requests per hour.
- **Throttling by API key:** Limit the number of requests from a specific API key to 500 requests per hour.
- **Blocking failed attempts:** If an API key is missing (indicating an invalid or missing key), that IP gets blocked after repeated failed attempts.

---

### 2. **Logging API Requests**

Next, let’s implement **logging** so you can track both successful and failed API requests.

#### 2.1. **Create a Custom Logger**

You can log all API requests, whether they’re successful or failed. This will give you insight into usage patterns, failed attempts, and general API traffic.

Create a logger initializer:

```bash
# config/initializers/api_logger.rb
API_LOGGER = Logger.new(Rails.root.join('log', 'api.log'))
API_LOGGER.level = Logger::INFO
```

This creates a new log file called `api.log` where API interactions will be stored.

---

#### 2.2. **Log Requests in the Controller**

Now, let’s log successful and failed API requests in the `Api::BaseController`. We’ll log:

- IP address
- API Key used
- Request path
- Result (success/failure)

```ruby
class Api::BaseController < ActionController::API
  before_action :authenticate_api_key!
  after_action :log_request

  private

  def authenticate_api_key!
    client_key = request.headers['X-API-KEY']
    valid_key = Rails.application.credentials.api_key || ENV['API_KEY']

    unless client_key && ActiveSupport::SecurityUtils.secure_compare(client_key, valid_key)
      log_failed_attempt
      render json: { error: 'Unauthorized' }, status: :unauthorized
    end
  end

  def log_request
    # Log all successful API requests
    API_LOGGER.info({
      time: Time.now,
      ip: request.remote_ip,
      api_key: request.headers['X-API-KEY'],
      path: request.fullpath,
      status: response.status
    }.to_json)
  end

  def log_failed_attempt
    # Log failed attempts
    API_LOGGER.warn({
      time: Time.now,
      ip: request.remote_ip,
      api_key: request.headers['X-API-KEY'],
      path: request.fullpath,
      status: 'failed'
    }.to_json)
  end
end
```

---

### 3. **Throttling Response**

When throttling kicks in (e.g., too many requests from a given IP or API key), `rack-attack` will automatically respond with:

- `429 Too Many Requests`
- And an optional response body, indicating the user has exceeded their rate limit.

This is automatically handled by `rack-attack`, so no additional code is needed.

---

### 4. **Optional: Add Custom Headers for Throttling Info**

You can also add custom headers in the response, which will give clients information about how many requests they have remaining:

```ruby
class Api::BaseController < ActionController::API
  before_action :authenticate_api_key!
  after_action :log_request

  def authenticate_api_key!
    client_key = request.headers['X-API-KEY']
    valid_key = Rails.application.credentials.api_key || ENV['API_KEY']

    unless client_key && ActiveSupport::SecurityUtils.secure_compare(client_key, valid_key)
      log_failed_attempt
      render json: { error: 'Unauthorized' }, status: :unauthorized
    end
  end

  def log_request
    # Log all successful API requests
    API_LOGGER.info({
      time: Time.now,
      ip: request.remote_ip,
      api_key: request.headers['X-API-KEY'],
      path: request.fullpath,
      status: response.status
    }.to_json)

    # Include rate limit info in the headers
    rate_limit = request.env['rack.attack.throttle_data']['api/ip']
    remaining_requests = rate_limit[:remaining]
    reset_time = Time.at(rate_limit[:epoch] + rate_limit[:period])

    response.headers['X-RateLimit-Remaining'] = remaining_requests.to_s
    response.headers['X-RateLimit-Reset'] = reset_time.to_s
  end
end
```

---

### 5. **Monitor Throttling and Logging**

You can now monitor the `api.log` file for request logs and failures. Additionally, use any **log aggregation service** (like **Loggly**, **Datadog**, or **Splunk**) to aggregate logs for easier monitoring in real-time.

---

## 📜 Summary: Full Throttling + Logging Setup

### 1. **Throttling with `rack-attack`**
- Protects against overuse of your API by IP and/or API key
- Automatically returns `429 Too Many Requests` when limits are exceeded

### 2. **Logging**
- Logs both successful and failed requests (including IP, API key, and request path)
- Helps with security audits, debugging, and identifying abuse patterns

### 3. **Optional Rate Limit Headers**
- Sends custom headers to the client about their rate limit status (`X-RateLimit-Remaining`, `X-RateLimit-Reset`)

---

You now have a **secure, monitored, and well-protected** Rails API!

Let me know if you need help with configuring or debugging any of this, or if you’d like further optimizations like **log rotation**, **custom metrics**, or integrating with a monitoring service.