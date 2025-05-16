Great! Let's break down **exactly how** this Ruby metaprogramming code works, line by line — both what it does and **how Ruby interprets it internally**.

---

### 🧱 Code:

```ruby
class User
  [:name, :email].each do |attr|
    define_method("get_#{attr}") do
      instance_variable_get("@#{attr}")
    end
  end
end
```

---

### 🔍 Step-by-Step Explanation

---

#### 🔹 `class User`

You’re defining a class called `User`.

> Ruby immediately starts evaluating the block inside `class User`.

---

#### 🔹 `[:name, :email].each do |attr|`

This iterates over an array of symbols: `:name` and `:email`.

---

#### 🔹 `define_method("get_#{attr}") do ... end`

You’re **creating methods dynamically**.

Let’s see what this does during each loop:

##### 📍 First iteration (`attr = :name`)

```ruby
define_method("get_name") do
  instance_variable_get("@name")
end
```

So Ruby defines a method called `get_name` that:

* Uses `instance_variable_get("@name")` to return the value of the `@name` variable.

##### 📍 Second iteration (`attr = :email`)

```ruby
define_method("get_email") do
  instance_variable_get("@email")
end
```

✅ Now your `User` class has two methods: `get_name` and `get_email`.

---

### 🔧 How does `define_method` work?

* `define_method` is a **private method of `Module`** (which `Class` inherits from).
* It takes a method name (as a symbol or string) and a block.
* Ruby adds this method to the class’s **method table** at runtime.

So the class ends up like this (roughly):

```ruby
class User
  def get_name
    instance_variable_get("@name")
  end

  def get_email
    instance_variable_get("@email")
  end
end
```

Except it was generated dynamically.

---

### ✅ Instance Part:

```ruby
u = User.new
u.instance_variable_set("@name", "Alice")
puts u.get_name  # => "Alice"
```

#### 🔹 `u = User.new`

Creates a new instance of `User`.

#### 🔹 `u.instance_variable_set("@name", "Alice")`

Dynamically sets the value of the instance variable `@name` to `"Alice"`.

> Think: `@name = "Alice"` but done dynamically.

#### 🔹 `u.get_name`

Calls the dynamically defined method `get_name`, which retrieves `@name` via `instance_variable_get`.

So this line prints:

```
Alice
```

---

### 🔁 Internals Recap

| Part                    | Internally                                   |
| ----------------------- | -------------------------------------------- |
| `define_method`         | Adds method to method table at runtime       |
| `instance_variable_get` | Reads raw value of an instance variable      |
| `instance_variable_set` | Sets instance variable by name               |
| `"@#{attr}"`            | String interpolation to create variable name |
| `User.new`              | Instantiates an object with empty state      |

---

### 🧠 Why Use This?

You use metaprogramming like this when you:

* Have **repetitive patterns** (e.g., getters/setters)
* Need to define behavior based on config/data
* Want to build **DSLs** or reduce boilerplate

---

Would you like a version that adds setters too (like `set_name`)?

Or maybe a live example that generates methods from a YAML or JSON config?

Let me know and I’ll build it out!

Absolutely! One of the most **practical use cases for Ruby metaprogramming** is dynamically defining methods based on **external configuration files** — like YAML or JSON. This is often seen in frameworks like Rails for things like form builders, dynamic attributes, etc.

---

## 🧱 Project Goal

We'll use a YAML file to define user attributes, and then generate **getter/setter methods** for each attribute using metaprogramming.

---

## 📁 Project Structure

```
dynamic_user/
├── user.rb           # Ruby class using metaprogramming
├── user_config.yml   # YAML config defining attributes
├── main.rb           # Entry point to run the logic
```

---

### 📄 1. `user_config.yml` – Config-Driven Data

```yaml
attributes:
  - name
  - email
  - age
```

---

### ✏️ 2. `user.rb` – Dynamically Build Class

```ruby
require 'yaml'

class User
  def initialize(config_file)
    config = YAML.load_file(config_file)

    config["attributes"].each do |attr|
      self.class.send(:define_method, attr) do
        instance_variable_get("@#{attr}")
      end

      self.class.send(:define_method, "#{attr}=") do |value|
        instance_variable_set("@#{attr}", value)
      end
    end
  end
end
```

🧠 This dynamically adds **getter and setter** methods for every attribute in the YAML file.

---

### ▶️ 3. `main.rb` – Run It

```ruby
require_relative './user'

user = User.new("user_config.yml")
user.name = "Alice"
user.email = "alice@example.com"
user.age = 28

puts user.name    # => Alice
puts user.email   # => alice@example.com
puts user.age     # => 28
```

---

## ✅ What’s Happening Behind the Scenes

* The YAML file is parsed into a Ruby hash.
* For each attribute:

  * `define_method(attr)` creates a method like `name`, `email`, etc.
  * `define_method("#{attr}=")` creates setters like `name=`, `email=`, etc.
* These methods are added **at runtime** based on config data.

---

## 🧠 Why This Is Powerful

* The class is completely **driven by config** — no hardcoded fields.
* Works well for dynamic form models, APIs, schema-less data, CMS fields, etc.
* You can update functionality by simply editing the YAML.

---

## 🧪 Bonus: JSON Version

If you'd rather use JSON:

```json
{
  "attributes": ["username", "password"]
}
```

Change `YAML.load_file(config_file)` to:

```ruby
require 'json'
config = JSON.parse(File.read(config_file))
```

---

Would you like:

* 🧰 Validation added to these dynamic attributes?
* 🧪 Type casting (e.g., make `age` an integer)?
* 💾 Saving these dynamic objects to a file or database?

Let me know — we can extend this into a dynamic model system or even a simple CMS!
