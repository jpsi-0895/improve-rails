In a **Ruby on Rails** application, the **`config/initializers/`** directory is used to define **initialization code** that runs when your application boots — just after the Rails framework is loaded, but **before** any web requests are handled.

---

## 🧠 What Are Rails Initializers?

* Files in `config/initializers/` are **automatically loaded** when the app starts.
* You use them to **set configuration**, **customize third-party gems**, or **initialize constants**, services, etc.

---

## 📁 Example: `config/initializers/`

Here are some common uses:

### 1. **Set a Global Constant**

```ruby
# config/initializers/constants.rb
APP_NAME = "MyCoolApp"
```

### 2. **Configure a Gem**

```ruby
# config/initializers/kaminari_config.rb
Kaminari.configure do |config|
  config.default_per_page = 25
end
```

### 3. **Set Time Zone or Encoding (if not in application.rb)**

```ruby
# config/initializers/time_zone.rb
Time.zone = "UTC"
```

### 4. **Logger Setup**

```ruby
# config/initializers/logger.rb
Rails.logger = Logger.new(STDOUT)
```

### 5. **Load Custom Libraries or Patches**

```ruby
# config/initializers/custom_patch.rb
require Rails.root.join("lib", "patches", "my_patch")
```

---

## 🔁 When Are Initializers Run?

They are executed **when the Rails environment loads**, so they **run once** when the app boots (in dev, test, or prod).

---

## 🛠 Best Practices

* Group related settings into one file.
* Avoid placing long-running or IO-heavy logic in initializers — it can slow down boot time.
* Use initializers for **configuration, not application logic**.

---

Would you like help writing a specific initializer (e.g., for a gem or environment config)?
