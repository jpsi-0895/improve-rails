Got it! 🔐 You want to **protect your Open API** (like a Rails JSON or GraphQL API) so that it **can't be freely called by anyone**, even though it's technically exposed.

Here's a breakdown of the best ways to **secure your Rails API** — depending on your needs:

---

## ✅ 1. **API Key Authentication (Simple & Effective)**

### 💡 Use case:
- Public-facing API
- You want to allow trusted clients (mobile apps, partners)
- Easy to manage without OAuth complexity

### 🛡 How to implement:

#### ✅ In your controller:

```ruby
class Api::BaseController < ActionController::API
  before_action :authenticate_api_key!

  private

  def authenticate_api_key!
    api_key = request.headers['X-API-KEY']
    unless api_key && ActiveSupport::SecurityUtils.secure_compare(api_key, ENV['API_KEY'])
      render json: { error: 'Unauthorized' }, status: :unauthorized
    end
  end
end
```

Then set `API_KEY` in your `.env` or credentials.

#### ✅ In your client:

```http
GET /api/v1/products
X-API-KEY: your-secret-key-here
```

---

## ✅ 2. **JWT (JSON Web Tokens)**

### 💡 Use case:
- Mobile clients, SSR apps, SPAs
- You want stateless auth (no sessions or cookies)

Use gems like:
```ruby
gem 'jwt'
```

**How it works:**
- Clients authenticate (email + password)
- You return a signed token
- They send it in headers for future calls
- You verify it on every request

🔐 Example:

```ruby
Authorization: Bearer eyJhbGciOiJIUzI1...
```

---

## ✅ 3. **OAuth2 (Doorkeeper)**

Use [`doorkeeper`](https://github.com/doorkeeper-gem/doorkeeper) gem for full-blown OAuth2 if you're building a **public developer API** or integrating **third-party apps**.

This is overkill for small APIs, but powerful for real authorization flows.

---

## ✅ 4. **Signed URLs (for limited access)**

For one-time, expiring API calls (e.g., downloading a file, accessing data):

```ruby
signed_url = Rails.application.routes.url_helpers.your_api_path(
  token: generate_signed_token(user.id, expires: 10.minutes)
)
```

Verify the token in the controller before allowing access.

---

## ✅ 5. **IP Whitelisting with `rack-attack`**

Block unknown clients at the **middleware level**:

```ruby
# config/initializers/rack_attack.rb
Rack::Attack.safelist('allow trusted IPs') do |req|
  ['1.2.3.4', '5.6.7.8'].include?(req.ip)
end

Rack::Attack.blocklist('block others') do |req|
  !['1.2.3.4', '5.6.7.8'].include?(req.ip)
end
```

---

## ✅ 6. **CORS Restrictions (for browser-based API use)**

If your API is exposed over the browser (like with React or Vue):

```ruby
# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'https://your-frontend.com'
    resource '*', headers: :any, methods: [:get, :post]
  end
end
```

✅ Blocks unauthorized browser calls

❌ Does not protect server-to-server calls (use API key or JWT for that)

---

## 🔐 Summary: Choose Your Strategy

| Security Level     | Method                | Best For                    |
|--------------------|-----------------------|-----------------------------|
| 🔓 Low             | Open API              | Internal apps/dev only      |
| 🔑 Medium          | API Key               | Simple server-to-server     |
| 🔐 High            | JWT                   | Mobile, SPA, secure access  |
| 🧾 Enterprise      | OAuth (Doorkeeper)    | 3rd party integrations      |
| 🔒 Extra           | IP Whitelist + CORS   | Private API + browser apps  |

---

Let me know:
- Who needs to access your API (mobile app, internal team, external clients)?
- Do you need login-based access, or just client verification?

I can give you a plug-and-play version of whichever auth style fits your case!