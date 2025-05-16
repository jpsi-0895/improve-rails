Great question! Let's break down **how metaprogramming works in Ruby** — in a way that's practical and clear.

---

## 🔁 What is Metaprogramming?

> **Metaprogramming is writing code that writes or manipulates other code at runtime.**

Ruby makes this easy because:

* **Everything is an object**
* **Classes and methods are open** and modifiable at runtime
* It supports **reflection**, **introspection**, and **dynamic code execution**

---

## 🔧 Core Mechanisms Behind Metaprogramming

Here’s how metaprogramming actually works under the hood:

---

### 1. ✅ **Open Classes (Monkey Patching)**

You can reopen any class and change it—even core ones like `String`:

```ruby
class String
  def shout
    self.upcase + "!"
  end
end

puts "hello".shout  # => "HELLO!"
```

**How?**
Ruby doesn’t lock down class definitions. Classes are just objects and can be redefined or extended on the fly.

---

### 2. 🧱 **`define_method` – Create Methods Dynamically**

```ruby
class User
  [:name, :email].each do |attr|
    define_method("get_#{attr}") do
      instance_variable_get("@#{attr}")
    end
  end
end

u = User.new
u.instance_variable_set("@name", "Alice")
puts u.get_name  # => "Alice"
```

**How?**
Ruby stores methods as symbols in an internal method table. `define_method` inserts new method definitions into this table at runtime.

---

### 3. 🚦 **`method_missing` – Catch Undefined Method Calls**

```ruby
class Magic
  def method_missing(method, *args)
    puts "You called: #{method} with args: #{args}"
  end
end

Magic.new.hello("world")  # => You called: hello with args: ["world"]
```

**How?**
If Ruby doesn’t find a method in the method table, it calls `method_missing`. You can override it to catch dynamic calls.

---

### 4. 🔍 **Reflection & Introspection**

You can ask Ruby about its own structure:

```ruby
"hi".class             # => String
String.instance_methods.include?(:upcase)  # => true
```

**How?**
Ruby objects contain metadata about their class, methods, variables, etc., accessible via methods like `.methods`, `.instance_variables`, `.class`, etc.

---

### 5. 📜 **`eval`, `class_eval`, `instance_eval` – Execute Code Strings or Blocks**

```ruby
code = "2 + 2"
puts eval(code)  # => 4

class Dog; end

Dog.class_eval do
  def bark
    "woof"
  end
end

puts Dog.new.bark  # => "woof"
```

**How?**
Ruby interprets the string/block as code in the current context and injects it into the running program.

---

## 💡 Summary Table

| Technique            | Purpose                            |
| -------------------- | ---------------------------------- |
| `define_method`      | Create methods at runtime          |
| `method_missing`     | Handle undefined methods           |
| `send`               | Call method by name                |
| `eval`, `class_eval` | Execute code dynamically           |
| `respond_to?`        | Introspection (does method exist?) |
| `singleton_class`    | Add method to one object           |

---

## 🧪 How Does It Work Internally?

Under the hood:

* Ruby stores classes and methods in tables.
* When you call `define_method` or `class_eval`, Ruby adds entries into these tables.
* When you call `method_missing`, it bypasses the normal lookup path.
* Reflection works by walking Ruby's internal object model and querying its metadata.

---

## 🧠 Real World Use Cases

| In Ruby/Rails | How Metaprogramming Helps                     |
| ------------- | --------------------------------------------- |
| ActiveRecord  | Defines methods like `user.name=` dynamically |
| RSpec         | DSL like `describe`, `it`                     |
| Rails Routing | `get '/home', to: 'home#index'`               |
| Gems          | Auto-generate accessors, methods              |

---

Would you like:

* A visual diagram of method lookup/metaprogramming?
* Live examples (IRB-based) to try step by step?
* Performance caveats or how to avoid misuse?

Let me know how deep you want to go!
