Let's configure **Sidekiq** and create a **specific job** example — both for **sending emails** and **syncing products**. This gives you a full setup for background processing in your Rails app.

---

## ✅ Step 1: Install & Configure Sidekiq

### Add to `Gemfile`

```ruby
# Gemfile
gem 'sidekiq'
```

Then install:

```bash
bundle install
```

---

### Set Sidekiq as Active Job adapter

In `config/application.rb` or environment-specific config:

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq
```

---

### Create Sidekiq Configuration

Create a file: `config/sidekiq.yml`

```yaml
:concurrency: 5
:queues:
  - default
  - mailers
  - low_priority
```

---

### Start Redis + Sidekiq

```bash
# Start Redis
redis-server

# Start Sidekiq
bundle exec sidekiq -C config/sidekiq.yml
```

---

## ✅ Step 2: Mount Sidekiq Dashboard

In `config/routes.rb`:

```ruby
require 'sidekiq/web'
mount Sidekiq::Web => '/sidekiq'
```

Then visit: `http://localhost:3000/sidekiq`

---

## ✉️ Example 1: Send Welcome Email with a Job

### Mailer:

```bash
rails generate mailer UserMailer
```

```ruby
# app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
  def welcome_email(user)
    @user = user
    mail(to: @user.email, subject: 'Welcome!')
  end
end
```

---

### Background Job:

```bash
rails generate job SendWelcomeEmail
```

```ruby
# app/jobs/send_welcome_email_job.rb
class SendWelcomeEmailJob < ApplicationJob
  queue_as :mailers

  def perform(user_id)
    user = User.find(user_id)
    UserMailer.welcome_email(user).deliver_now
  end
end
```

---

### Trigger Email in Controller or Callback:

```ruby
SendWelcomeEmailJob.perform_later(@user.id)
```

---

## 🔄 Example 2: Sync Products to External API

```bash
rails generate job SyncProducts
```

```ruby
# app/jobs/sync_products_job.rb
class SyncProductsJob < ApplicationJob
  queue_as :default

  def perform
    products = Product.all
    products.each do |product|
      # Simulate external API sync
      ExternalApi.sync_product(product)
    end
  end
end
```

---

### External API Module (Mock):

```ruby
# app/services/external_api.rb
module ExternalApi
  def self.sync_product(product)
    # Placeholder for real HTTP call
    Rails.logger.info("Syncing product #{product.id} to external service.")
  end
end
```

---

### Trigger Sync Job:

```ruby
SyncProductsJob.perform_later
```

You can also schedule it using [Sidekiq-Cron](https://github.com/ondrejbartas/sidekiq-cron) if needed.

---

Would you like to schedule product sync jobs at intervals (e.g., every hour) using `sidekiq-cron` or `whenever` gem?
