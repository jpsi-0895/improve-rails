**Bug analysis in Rails** involves identifying, investigating, and resolving unexpected behavior, crashes, or logical errors in a Rails application. A structured approach helps you isolate the root cause efficiently.

---

## 🧭 Step-by-Step Bug Analysis in Rails

### 1. 🧾 **Check Logs**

Start with Rails logs—they give real-time error traces and stack traces.

#### Default log files:
- `log/development.log`
- `log/production.log`
- Use `tail` to monitor:
  ```bash
  tail -f log/development.log
  ```

Check for:
- Exceptions (e.g., `NoMethodError`, `ActiveRecord::RecordNotFound`)
- Request parameters
- SQL queries
- Background job failures

---

### 2. 🔍 **Use `byebug` or `pry` for Debugging**

Insert a breakpoint inside your code where you suspect the bug:

```ruby
byebug  # or binding.pry
```

Then run the app and trigger the bug. You'll get an interactive console in the terminal to inspect variables and flow.

#### Example:
```ruby
def show
  @product = Product.find(params[:id])
  byebug
end
```

Use commands like:
- `next` – next line
- `step` – step into
- `continue` – resume
- `params`, `@product`, etc. – inspect variables

---

### 3. 🧪 **Write Tests to Reproduce the Bug**

Write a unit, functional, or integration test that fails due to the bug. This prevents regressions later and helps isolate the conditions causing the bug.

```ruby
it "raises error when product is missing" do
  expect { get :show, params: { id: -1 } }.to raise_error(ActiveRecord::RecordNotFound)
end
```

---

### 4. 🔐 **Check for External Factors**

Some bugs come from:
- Misconfigured environment variables
- Third-party API failures
- Database inconsistencies
- SSL or proxy misconfigurations (especially in production)

---

### 5. 🔬 **Use Error Trackers (for Production)**

For automated error reporting and analysis in production, use tools like:

- **Sentry**
- **Bugsnag**
- **Rollbar**
- **Honeybadger**

These tools provide:
- Stack traces
- Request data
- User context
- Deploy tracking

#### Example (Sentry):
```ruby
Sentry.capture_exception(exception)
```

---

### 6. 🔐 **Check for Silent Failures**

- Rescue blocks swallowing exceptions (`rescue nil`, `rescue => e; logger.error e`)
- Use `Rails.logger.debug`, `warn`, `error` to trace unexpected behavior

---

### 7. 🧹 **Use `rails console` to Inspect Models**

Test model logic, scopes, and queries directly:

```bash
rails console
```

Then try:
```ruby
User.find(999)  # See what error occurs
Order.where(status: nil).count  # Check data issues
```

---

### 8. 🧰 Useful Tools

| Tool | Purpose |
|------|---------|
| `rails server --debugger` | Starts server with debugging support |
| `rails dbconsole`         | Inspect DB records directly |
| `rails routes`            | Check route misconfigurations |
| `tail -f log/production.log` | Monitor real-time logs |
| `rspec` / `minitest`      | Run test suite to reproduce bug |

---

## ✅ Best Practices

- Reproduce locally if possible
- Isolate minimal failing code
- Create a failing test
- Fix, then ensure test passes
- Watch production errors via monitoring tools

---

Would you like help setting up a debugger, writing a test for a specific bug, or integrating an error tracking tool like Sentry?