### ✅ Jobs in Rails (Background Jobs with Active Job)

Rails provides a built-in framework called **Active Job** to declare jobs and run them with a variety of backends (e.g., Sidekiq, Delayed Job, Resque).

---

## 🧱 1. Create a Job

You can generate a job using Rails generator:

```bash
rails generate job MyTask
```

This creates:

```ruby
# app/jobs/my_task_job.rb
class MyTaskJob < ApplicationJob
  queue_as :default

  def perform(*args)
    # Do something later
    Rails.logger.info("Running job with args: #{args.inspect}")
  end
end
```

You can then enqueue it:

```ruby
MyTaskJob.perform_later('some_argument')
```

Or run it inline (for testing):

```ruby
MyTaskJob.perform_now('some_argument')
```

---

## ⚙️ 2. Configure Job Backend

### Development / Test

Default backend is `:async`:

```ruby
# config/environments/development.rb
config.active_job.queue_adapter = :async
```

### Production (Recommended: Sidekiq)

Add to Gemfile:

```ruby
gem 'sidekiq'
```

Then set in `config/application.rb`:

```ruby
config.active_job.queue_adapter = :sidekiq
```

And run Sidekiq:

```bash
bundle exec sidekiq
```

---

## 🛠 Example: Send Email in Background

```ruby
# app/jobs/welcome_email_job.rb
class WelcomeEmailJob < ApplicationJob
  queue_as :mailers

  def perform(user_id)
    user = User.find(user_id)
    UserMailer.welcome_email(user).deliver_now
  end
end
```

Enqueue it:

```ruby
WelcomeEmailJob.perform_later(user.id)
```

---

## 📊 Job Monitoring (Sidekiq UI)

Mount in `routes.rb`:

```ruby
require 'sidekiq/web'
mount Sidekiq::Web => '/sidekiq'
```

Visit `/sidekiq` in your browser to monitor jobs.

---

Would you like me to help you configure Sidekiq or create a specific job, like sending emails or syncing products?
