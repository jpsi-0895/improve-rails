Securing an **Open API** in **Rails** depends on how “open” you want it to be—public but rate-limited? Authenticated with API keys or OAuth? Here's a breakdown of **common patterns to secure public-facing APIs in Rails**, starting from lightweight to enterprise-grade.

---

## 🔐 Securing a Public/Open API in Rails

---

### ✅ 1. **API Key Authentication (Simple & Secure)**

Best if you want to give trusted partners or apps access to your API.

#### a) Add an `api_key` column to `User` or `ApiClient` model:

```bash
rails g model ApiClient name:string api_key:string
rake db:migrate
```

Generate a secure token on creation:

```ruby
before_create :generate_api_key

def generate_api_key
  self.api_key = SecureRandom.hex(32)
end
```

---

#### b) Authenticate in your API controller:

```ruby
class Api::BaseController < ActionController::API
  before_action :authenticate_api_key!

  private

  def authenticate_api_key!
    token = request.headers['Authorization']&.split(' ')&.last
    @current_client = ApiClient.find_by(api_key: token)

    render json: { error: 'Unauthorized' }, status: :unauthorized unless @current_client
  end
end
```

💡 **Use `Bearer <api_key>`** in the `Authorization` header.

---

### ✅ 2. **Rate Limiting (Throttle Public Access)**

Use `rack-attack` gem to prevent abuse.

```ruby
# Gemfile
gem 'rack-attack'
```

In `config/initializers/rack_attack.rb`:

```ruby
class Rack::Attack
  # Limit requests to 100 per IP per 5 minutes
  throttle('req/ip', limit: 100, period: 5.minutes) do |req|
    req.ip
  end
end
```

Enable middleware in `application.rb`:

```ruby
config.middleware.use Rack::Attack
```

Optional: use Redis for more advanced tracking.

---

### ✅ 3. **OAuth 2.0 / OpenID Connect (For SSO or 3rd-party apps)**

For securing with user login or 3rd-party app tokens, use:

- `doorkeeper` gem (for full OAuth 2.0 provider)
- `omniauth` (for login via external IdPs)

Install `doorkeeper`:

```ruby
gem 'doorkeeper'
```

Then:

```bash
rails generate doorkeeper:install
rails db:migrate
```

Now you can issue access tokens via `/oauth/token` and protect APIs with:

```ruby
before_action :doorkeeper_authorize!
```

You can define scopes, grant types, etc.

---

### ✅ 4. **CORS Protection**

For Open APIs accessible via browser-based clients, restrict access origins using `rack-cors`.

```ruby
# Gemfile
gem 'rack-cors'
```

```ruby
# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'your-frontend-domain.com'
    resource '*',
      headers: :any,
      methods: [:get, :post, :patch, :put, :delete, :options]
  end
end
```

---

### ✅ 5. **HTTPS Only**

Force SSL in production:

```ruby
# config/environments/production.rb
config.force_ssl = true
```

---

### ✅ 6. **Audit, Logs, and Alerts**

- Use `ActiveSupport::Notifications` to log API activity.
- Add alerts on unusual behavior (spikes in traffic, failed auth, etc.)

---

### 🔐 Summary: Pick Based on Your Use Case

| Use Case                         | Recommended Securing Method         |
|----------------------------------|-------------------------------------|
| Internal / partner APIs          | API key + rate limiting             |
| Third-party integration          | OAuth2 with `doorkeeper`            |
| Public open API (read-only)      | Rate limit + CORS                   |
| Authenticated user access        | JWT or OIDC via `omniauth`          |

---

If you tell me more about **how open your API is**, or if you're building something like a **public product catalog, an internal microservice, or a B2B integration**, I can tailor this setup more specifically for you. Want to plug it into Swagger too?