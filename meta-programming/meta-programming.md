**Metaprogramming in Ruby** is one of its most powerful (and elegant) features — it allows you to write code that **writes or modifies other code at runtime**. This makes Ruby highly flexible and expressive, especially useful in DSLs (like Rails), gems, and dynamic logic.

---

## 🧠 What is Metaprogramming?

> Writing code that **generates**, **modifies**, or **executes** other code **at runtime**.

Ruby treats **classes, methods, and variables as objects**, so you can modify them dynamically.

---

## 📌 Common Techniques in Ruby Metaprogramming

---

### 1. **`define_method` — Create Methods at Runtime**

```ruby
class MyClass
  [:name, :email].each do |attr|
    define_method("get_#{attr}") do
      instance_variable_get("@#{attr}")
    end

    define_method("set_#{attr}") do |value|
      instance_variable_set("@#{attr}", value)
    end
  end
end

user = MyClass.new
user.set_name("Alice")
puts user.get_name  # => Alice
```

---

### 2. **`method_missing` — Catch Undefined Methods**

```ruby
class DynamicCaller
  def method_missing(method, *args, &block)
    puts "You tried calling #{method} with args #{args}"
  end
end

dc = DynamicCaller.new
dc.something("hello")  # => You tried calling something with args ["hello"]
```

> 🔥 Be careful! This can make debugging harder. Pair it with `respond_to_missing?`.

---

### 3. **`send` — Call Any Method by Name (Even Private)**

```ruby
class Secret
  def initialize
    @secret = "top_secret"
  end

  private

  def reveal
    "Secret is #{@secret}"
  end
end

s = Secret.new
puts s.send(:reveal)  # => "Secret is top_secret"
```

---

### 4. **`class_eval` / `instance_eval` — Inject Code Dynamically**

#### 🔹 `class_eval` — Add methods to a class:

```ruby
class Person; end

Person.class_eval do
  def greet
    "Hi!"
  end
end

puts Person.new.greet  # => "Hi!"
```

#### 🔹 `instance_eval` — Work with an object’s context:

```ruby
str = "Hello"
str.instance_eval do
  def shout
    upcase + "!"
  end
end

puts str.shout  # => HELLO!
```

---

### 5. **`define_singleton_method` — Add Method to One Object**

```ruby
dog = "Fido"
dog.define_singleton_method(:bark) { "Woof!" }

puts dog.bark  # => Woof!
```

---

## 🏗 Example: Simple DSL Using Metaprogramming

```ruby
class Config
  def self.setting(name, &block)
    define_method(name, &block)
  end
end

class AppConfig < Config
  setting :database do
    "postgresql"
  end
end

cfg = AppConfig.new
puts cfg.database  # => postgresql
```

---

## 🧠 When to Use Metaprogramming

✅ Use it for:

* DSLs (like RSpec, Rails migrations)
* Reducing repetitive code
* Dynamic behavior (e.g., APIs, models)

🚫 Avoid it when:

* Code clarity matters more than flexibility
* You’re doing something that could be solved more simply

---

## 📚 Real World Use in Rails

* **`attr_accessor`**, `before_action`, `has_many` — all are built with metaprogramming
* Rails DSL like:

  ```ruby
  validates :name, presence: true
  ```

---

## 🧪 Want to Try It Live?

Let me know if you’d like:

* A mini metaprogramming exercise
* A scaffolded Ruby DSL project
* An explanation of how Rails uses these behind the scenes

Metaprogramming is powerful — but like with magic, use it wisely 🪄
