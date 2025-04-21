Ah yep — you're getting the error:

> **"If this is a Rails app in API mode, you need to enable sessions"**

That happens because **Rails API-only apps** disable session & cookie middleware by default (to keep things lightweight), but **Sidekiq Web UI** depends on it to manage login/auth.

Let’s fix that in ✌️ easy steps.

---

## ✅ Step 1: Enable Sessions for Sidekiq Route

Edit your **`config/routes.rb`** like this:

```ruby
require 'sidekiq/web'

Rails.application.routes.draw do
  # Mount Sidekiq web UI with middleware for sessions
  mount Sidekiq::Web => '/sidekiq', constraints: ->(req) { true }

  # other routes...
end
```

Then wrap the mount in middleware manually if you're in API mode.

---

## ✅ Step 2: Manually Add Required Middleware (only for Sidekiq)

In `config/application.rb`, add this inside the `class Application < Rails::Application` block:

```ruby
# config/application.rb
module YourAppName
  class Application < Rails::Application
    # Enable cookies and sessions for Sidekiq Web UI in API mode
    config.middleware.use ActionDispatch::Cookies
    config.middleware.use ActionDispatch::Session::CookieStore, key: '_your_app_session'
  end
end
```

> 🔁 Replace `_your_app_session` with a custom name if you like.

---

## ✅ Step 3 (Optional): Secure Access in Production

You should restrict access to the `/sidekiq` route in production, e.g., with basic auth:

```ruby
Sidekiq::Web.use Rack::Auth::Basic do |username, password|
  ActiveSupport::SecurityUtils.secure_compare(username, ENV["SIDEKIQ_USER"]) &
  ActiveSupport::SecurityUtils.secure_compare(password, ENV["SIDEKIQ_PASS"])
end
```

Set `SIDEKIQ_USER` and `SIDEKIQ_PASS` in your environment variables.

---

## ✅ Final Step: Restart the Server

```bash
rails s
```

Now open:

```
http://localhost:3000/sidekiq
```

✅ And you should see the dashboard!

---

Let me know if you want to add authentication, protect in staging/prod, or schedule recurring jobs.